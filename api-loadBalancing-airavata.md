---
layout: default
---

# API Load Balancing with High Availability and Fault Tolerance

This exercise was done as a part of [Airavata Courses](http://courses.airavata.org/), the motive here is to propose solutions to Airavata's development bottle necks and implement proof of concept to help envision solutions.

This proposal is built to improve load balancing capabilities of Apache Airavata, API gateway is a part of Airavata's middleware service, providing the essential abstraction and security needed to several micro-services deployed as Component programming interfaces, Hence API gateway cannot be a single point of failure. Therefore, we propose a proof of concept to deploy API gateway in a distributed environment and setup a load balancer with dynamic service discovery.  

Please refer [Documentation](http://airavata.apache.org/learning.html) for more details about the architecture of Apache Airavata middleware.

## Source Code:
* [API-Server](https://github.com/airavata-courses/spring17-API-Server/tree/master)
* [Load-Balancer](https://github.com/airavata-courses/spring17-API-Server/tree/loadBalancer)
* [API-Server userdata script used in AutoScaling group](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/userdata.sh)
* [Load-Balancer userdata script used in AutoScaling group](https://github.com/airavata-courses/spring17-API-Server/blob/loadBalancer/build-scripts/userdata.sh)

## Lets start with the Software Defined Environment (SDE)

The Environment currently setup at AWS is described here,

![EnvWithLoadBalancer.png](https://github.com/airavata-courses/spring17-API-Server/blob/wiki-content/wiki-content/EnvWithLoadBalancer.png)

There are many parts to this environment setup, lets break them into parts:

1. **AutoScaling Group:**
* load-balancer: Here the auto-sclaing is used define the SDE, and also maintain the High Availability of the dedicated load balancing instance. As this is a Development Env, only one instance is used, but with minor tweak this group can up-scale to include more instances.
* API-Server: We have used the autoscaling feature of AWS to handle fault tolerance with our spot instances, Since the spot instances are prone to go offline at any time, we need to set up a fault tolerant SDE to handle these scenario's, i.e automatically bring up a new instance with all the data and configuration for the api-server to run on instance start-up, in case a spot instance goes offline.

2. **Launch Configuration:** Every Autoscaling group is ideally assigned to launch configuration, essentially providing the group with all the configuration and the data to bring up a new instance. We have provided a launch script ([userdata.sh](https://github.com/airavata-courses/spring17-API-Server/blob/master/build-scripts/userdata.sh)), which basically sets up the newly booted system by installing the essentially software components to build, deploy and run the API-Server.

3. **Pre baked AMI (Amazon Machine Image):** Why ? Coz we need to setup the newly booted spot instances to work with our already configured CICD pipeline, and to accomplish that, the VM's should have access to Amazon access keys, which are brewed into the custom Linux OS image. Also the AMI's are equipped with Developer's SSH public keys, which eliminates the need to manually setup key after new instances are spawned. This very setup is not the perfect, Ideally a secret management engine ([Vault](https://www.vaultproject.io/)) will be used in production environment.

4. **Amazon spot instances:** Spot instances have been used to keep the costs low, and also test the limits of our load balancer for api-gateway. The AutoScaling group will keep the instances alive until the bid price is less than the market price.

***

## Load Balancer setup:

Gone are the days when balancers were static and needed manual configuration, with new age solutions like [Fabio](https://github.com/eBay/Fabio) and [Consul](https://github.com/hashicorp/consul) providing dynamic configuration and automated service discovery, load balancing with high Availability and Fault Tolerance becomes a breeze. To top it of the whole enviroment is defined as code and hooked up to a CICD pipeline with Travis-CI, hence making any changes to the enviroment is just a github commit away.

**Lets discuss our configuration...**

1. [**Consul:**](https://github.com/hashicorp/consul)
* The main purpose of using consul is to provide Service discovery in our environment, as a part of our SDE, we install consul on all the instances including the load-balancer, forming a mesh network of nodes. Therefore every time a new instance is spawned by AutoScaling group, the instance will automatically register itself with the network.
* For Service registration, A Python script is used to call the consul API and register the API server end-point every few seconds. Hence providing Automated service discovery.
2. [**Fabio:**](https://github.com/eBay/Fabio) Fabio is a full featured zero-configuration load balancer specially built to handle micro-services deployment. It integrates well with consul and uses the consul service discovery to manage the configuration. Hence no restarts or manual configuration is required to setup the load balancer.

## Future Work:

* Making the software defined environment cloud independent.

[back](./)
