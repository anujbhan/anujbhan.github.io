---
layout: blog
---

---
# Comparing DevOps tools

![Iac image](https://cdn-images-1.medium.com/max/1000/1*LEGSC7cU0sh8dyzz0o5n9A.png)

At Apache Airavata, we are trying to zero down on the tool to use for Infrastructure-as-code, but first, let us kick-off the comparison by asking the question,

## Why Infrastructure-as-Code ?

Instead of clicking around a web UI or SSHing to a server and manually executing commands, the idea behind IAC is to write code to define, provision, and manage your infrastructure. This has a number of benefits:

 * You can automate your entire provisioning and deployment process, which makes it much faster and more reliable than any manual process.
 * You can represent the state of your infrastructure in source files that anyone can read rather than a sysadmin’s head.
 * You can store those source files in version control, which means the entire history of your infrastructure is now captured in the commit log, which you can use to debug problems, and if necessary, roll back to older versions.
 * You can validate each infrastructure change through code reviews and automated tests.
 * You can create reusable, documented, battle-tested infrastructure packages that make it easier to scale and evolve your infrastructure.

There is one other very important, and often overlooked reason for why you should use IAC: it makes developers happy. Deploying code is a repetitive and tedious task. A computer can do that sort of thing quickly and reliably, but a human will be slow and error prone.

## Tools comparison

![IaC tools chart](https://cdn-images-1.medium.com/max/2000/1*bVC97LGrOY4R5E4WwzMLzw.png)

Our main focus is on **Terraform vs Ansible**, both are compelling tools with client-only architectures, but they differ in many aspects, below are such considerations,
* **Configuration Management vs Orchestration**

  Chef, Puppet, Ansible, and SaltStack are all “configuration management” tools, which means they are designed to install and manage software on existing servers. CloudFormation and Terraform are “orchestration tools”, which means they are designed to provision the servers themselves, leaving the job of configuring those servers to other tools. These two categories are not mutually exclusive, as most configuration management tools can do some degree of provisioning and most orchestration tools can do some degree of configuration management. But the focus on configuration management or orchestration means that some of the tools are going to be a better fit for certain types of tasks.

* **Mutable Infrastructure vs Immutable Infrastructure**

  Configuration management tools such as Chef, Puppet, Ansible, and SaltStack typically default to a mutable infrastructure paradigm. For example, if you tell Chef to install a new version of OpenSSL, it’ll run the software update on your existing servers and the changes will happen in-place.

  If you’re using an orchestration tool such as Terraform to deploy machine images created by Docker or Packer, then every “change” is actually a deployment of a new server (just like every “change” to a variable in functional programming actually returns a new variable). For example, to deploy a new version of OpenSSL, you would create a new image using Packer or Docker with the new version of OpenSSL already installed, deploy that image across a set of totally new servers, and then destroy the old servers.

   Of course, it’s possible to force configuration management tools to do immutable deployments too, but it’s not the idiomatic approach for those tools, whereas it’s a natural way to use orchestration tools.

* **Procedural vs Declarative**

  Chef and Ansible encourage a procedural style where you write code that specifies, step-by-step, how to achieve some desired end state. Terraform, CloudFormation, SaltStack, and Puppet all encourage a more declarative style where you write code that specifies your desired end state, and the IAC tool itself is responsible for figuring out how to achieve that state.

  This becomes more relevant when the tool in consideration keep state (metadata) of infrastructure, for example, Terraform uses the state information to keep an eye on all the entities, and when an additional change is pushed, it compares it with existing metadata and only applies the delta changes to infrastructure. Hence reducing the time and efforts put my developer to maintain the inventory information.

## What did we Choose ?

In Case of Apache Airavata, we use multiple cloud vendors and most of the times, Airavata is deployed on University infrastructure. Therefore going with cloud specific tools like AWS Cloudformation or OpenStack HeatTemplates makes no sense. To top it of, the Apache Airavata middleware needs a lot of configuration setup inorder to get the application running, therefore a well footed Configuration management tool is necessory like Ansible or Chef.

![Terraform Logo & Ansible Logo]({{ site.url }}/images/ansible&terraform.png)

The conclusion is, Terraform provides provides an unmatched capabilities for cloud provisioning, specially in AWS and OpenStack, But Cannot provide that full fledged support for configuration management. Hence Terraform for provision and Ansible for configuration management is the way to go for Apache Airavata.

## GO To [HOME]({{ site.url }}/index)
