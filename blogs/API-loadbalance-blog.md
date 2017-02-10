---
layout: default
---

# API Load Balancing with High Availability and Fault Tolerance

***

## Problem Statement: API Gateway Load Balancing

This proof of concept is built to improve upon the load balancing capabilities of Apache Airavata, API gateway is a part of Airavata's middleware service, providing the essential abstraction and security needed to several micro-services deployed as Component programming interfaces. API gateway is deployed on a single instance in production, hence is the single point of failure for the whole middleware service, therefore this repository will serve as the test bench for trying out all the load balancing and infrastructural technologies and come up with a feasible solution for Airavata. Along with load-balancing, this project also serves the infrastructural needs associated with the solution. Please refer [Airavata Docs](http://airavata.apache.org/learning.html) for more details about the architecture.

***

## Source Code:
  * [API-Server](https://github.com/airavata-courses/spring17-API-Server/tree/master)

  * [Load-Balancer](https://github.com/airavata-courses/spring17-API-Server/tree/loadBalancer)

  * [API-Server userdata script used in AutoScaling group](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/userdata.sh)

  * [Load-Balancer userdata script used in AutoScaling group](https://github.com/airavata-courses/spring17-API-Server/blob/loadBalancer/build-scripts/userdata.sh)

  * [Terraform files](https://github.com/airavata-courses/spring17-API-Server/tree/master/build-scripts/terraform)

***

## Lets start with the Software Defined Environment (SDE)

The Environment currently setup at AWS is described here,

![EnvWithLoadBalancer block diagram]({{ site.url }}/images/EnvWithLoadBalancer.png)

There are many parts to this environment setup, lets break them into parts:

**AutoScaling Group:**

1. **load-balancer:** Here the autoscaling is used define the SDE, and also maintain the High Availability of the dedicated load balancing instance. As this is a Development Env, only one instance is used, but with minor tweak this group can up-scale to include more instances.

2. **API-Server:** We have used the autoscaling feature of AWS to handle fault tolerance with our spot instances, Since the spot instances are prone to go offline at any time, we need to set up a fault tolerant SDE to handle these scenario's, i.e automatically bring up a new instance with all the data and configuration for the api-server to run on instance start-up, in case a spot instance goes offline.

**Launch Configuration:** Every Autoscaling group is ideally assigned to launch configuration, essentially providing the group with all the configuration and the data to bring up a new instance. We have provided a launch script ([userdata.sh](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/userdata.sh)), which basically sets up the newly booted system by installing the essentially software components to build, deploy and run the API-Server.

**Pre baked AMI (Amazon Machine Image):** Why ? Coz we need to setup the newly booted spot instances to work with our already configured CICD pipeline, and to accomplish that, the VM's should have access to Amazon access keys, which are brewed into the custom Linux OS image. Also the AMI's are equipped with Developer's SSH public keys, which eliminates the need to manually setup key after new instances are spawned. This very setup is not the perfect, Ideally a secret management engine ([Vault](https://www.vaultproject.io/)) will be used in production environment.

**Amazon spot instances:** Spot instances have been used to keep the costs low, and also test the limits of our load balancer for api-gateway. The AutoScaling group will keep the instances alive until the bid price is less than the market price.

***

### Now lets discuss how we configured this setup

In our first iteration of development, we used the AWS console to configure the infrastructure, but upon further discussion with our peers, we released that all this setup is AWS specific and managing the whole technology was a bit difficult, specially changing any entity would consume a lot of time. Hence, A abstraction layer was needed which could mask the details of specific cloud vendor and provide a unified platform to manage the whole infrastructure. we discovered "**Terraform**".

[Terraform](https://www.terraform.io/), a cloud agnostic tool to manage infrastructure for multiple cloud vendors, basically, providing a unified platform to create, change, evolve and destroy infrastructure with ease. For more details about Terraform's features and capabilities, visit their [Docs](https://www.terraform.io/docs/index.html).

Now, concentrating on the cloud agnostic behaviors, I agree that the infrastructure currently setup is still AWS specific, but on the positive side, every other cloud provider offer these features, for example Google provides "Managed instance groups" feature which is the same thing as "AutoScaling groups" in Amazon web Services and Google cloud's "CodeShip" feature is the same as "CodeDeploy" in AWS. Therefore introducing Terraform in our SDE enables the project to be hosted on multiple providers, and all this by writing a simple JSON file like the [one](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/terraform/spring17-api-infra.tf) we wrote.

Lets examine our terraform [files](https://github.com/airavata-courses/spring17-API-Server/tree/master/build-scripts/terraform) :

1. [spring17-api-infra](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/terraform/spring17-api-infra.tf) : This is the main file where the actual infrastructure is defined, for example, creating an autoscaling group in AWS is as simple as this,

                  ```
                  resource "aws_autoscaling_group" "web-asg" {
                    availability_zones   = ["${split(",", var.availability_zones)}"]
                    name                 = "spring17-api-terraform-asg"
                    max_size             = "${var.asg_max}"
                    min_size             = "${var.asg_min}"
                    desired_capacity     = "${var.asg_desired}"
                    force_delete         = true
                    launch_configuration = "${aws_launch_configuration.web-lc.name}"

                    # Needed, to enable modification to launch config post creation
                    lifecycle {
                        create_before_destroy = true
                      }

                    #vpc_zone_identifier = ["${split(",", var.availability_zones)}"]
                    tag {
                      key                 = "Name"
                      value               = "spring17-api-terraform-asg-instances"
                      propagate_at_launch = "true"
                    }
                  }
                  ```
2. [variables.tf](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/terraform/variables.tf) : This file contains the infrastructure variables like the desired number of instances in AutoScaling groups,

                  ```
                  variable "asg_desired" {
                    description = "Desired numbers of servers in ASG"
                    default     = "2"
                  }
                  ```
3. [output.tf](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/terraform/outputs.tf) : This file contains all the out variable which will be printed upon successful creation of entities, for example,

                  ```
                  output "security_group" {
                    value = "${aws_security_group.default.id}"
                  }
                  ```
4. [Terraform.fstate](https://www.terraform.io/docs/state/index.html): This file is maintained by terraform for versioning the infrastructure, or to rephrase, keeping track of our infrastructure, which is needed to make modifications, rollback, updating or deleting entities. Since the state contains all the confidential details about our infrastructure, we have not committed it to the version control, rather used the feature of [Remote State](https://www.terraform.io/docs/state/remote/index.html) provided by Terraform to safely put our files in s3 bucket.


***

## Load Balancer setup:

Gone are the days when balancers were static and needed manual configuration, with new age solutions like [Fabio](https://github.com/eBay/Fabio) and [Consul](https://github.com/hashicorp/consul) providing dynamic configuration and automated service discovery, load balancing with high Availability and Fault Tolerance becomes a breeze. To top it of the whole environment is defined as code and hooked up to a CICD pipeline with Travis-CI, hence making any changes to the environment is just a GitHub commit away.

**Here are the possible solutions for Load balancing:**

Our main concern when researching about this topic was to introduce dynamic and automated nature to load-balancing architecture, or to rephrase it, we were more concerned about service discovery and automated configuration of load balancer service. Hence below mentioned are the possible solutions,

   1. **1)** [Consul](https://www.consul.io/) for service discovery and [Fabio](https://github.com/eBay/fabio) for load balancing.

   2. **2)** [Consul](https://www.consul.io/) for service discovery and [consul template](https://github.com/hashicorp/consul-template) plus [HAProxy](http://www.haproxy.org/) for load balancing.

We tried our hand at both the solutions, our inference is, Since we are implementing load balancer for api-server and moreover our API-server uses [Apache Thrift](https://thrift.apache.org/) for API communications, i.e Our solution should load-balance TCP traffic, though both Fabio and HAProxy support this feature, we feel that, HAProxy provides more features and is easier to configure for TCP traffic.

**Lets discuss our configuration...**

1. [**Consul:**](https://github.com/hashicorp/consul) The main purpose of using consul is to provide Service discovery in our environment, as a part of our SDE, we install consul on all the instances including the load-balancer, forming a mesh network of nodes. Therefore every time a new instance is spawned by AutoScaling group, the instance will automatically register itself with the network.

2. We embedded some java code in our Thrift API service which calls the consul API to register itself to consul after successfully starting the server. This provides automated service discovery.

3. [Consul Template](https://github.com/hashicorp/consul-template): It is daemon running on load-balancer instance, which monitors consul for any changes in service configuration, and is used to update the HAProxy files and issue commands to HAProxy, hence enabling auto update for HAProxy.

4. [HAProxy](http://www.haproxy.org/): I don't think there is a lot to mention here, HAProxy is a reliable, stable TCP/HTTP/s load-balancer trusted by millions of users.

## GO To [HOME]({{ site.url }}/index)
