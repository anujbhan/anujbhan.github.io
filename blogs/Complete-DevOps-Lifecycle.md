---
layout: blog
---

![Apache Airavata](http://airavata.apache.org/assets/img/airavata-brand.png)

# __Complete DevOps LifeCycle for Apache Airavata__

## __Problem Statement__

Come up with DevOps solutions including but not limited to Infrastructure provisioning, Continuous integration and Continuous deployment which is cloud agnostic in nature. The solution offered should suit the distributed nature of Airavata deployments enabling the open source developer to develop, test and deploy new code with ease.

## __Possible Solutions__

DevOps, a portmanteau of development and operations, is quite the buzzword now. Unfortunately, development and operations are two organizational entities that tend not to get along very well. In this post, I will try to explain, why DevOps represents a mental shift from “us versus them” to a more cohesive, results-oriented approach.

Adopting DevOps culture in Apache Airavata is more crucial than ever, it will enable easy bootstrapping for new developers, hence bringing in more developers to open source community. Along with above-mentioned advantage, DevOps will help Airavata make faster progress from development to deployment stage, this is especially important to Airavata because, our middleware service is provided as Software-as-Service spanning multiple cloud vendors, including AWS and JetStream (OpenStack). Managing Integration and Deployment is a huge task, We look to adopt DevOps for reducing this workload by automated CI and CD pipelines.

This topic is being continuously brainstormed at the Airavata Course - Advanced Science Gateway, at Indiana University Bloomington. We have come up with a list of goals to achieve this milestone,

  *   **1.)** Local Provisioning for Development & Testing
  *   **2.)** CI & CD post-GitHub pull requests
  *   **3.)** Upgrading application on Dev, Staging and Production Environment
  *   **4.)** Debugging deployed code base

Presently, the DevOps adaptation is partial in Airavata but achieving the above-mentioned goals, though a ambitious move, will surely help Airavata achieve greater heights.

**Note:** This blog post is to showcase my thought process rather than giving out actual solutions, The solutions will be covered in subsequent blog posts, with a thorough comparison of approaches and tools used to achieve it.

Let's discuss the goals we wish to achieve with DevOps adaptations,

### Local provisioning for Development & Testing

Understanding a software system written by other developers and adding new features/modules to the code base is in itself a very arduous task, but is unavoidable. What comes next is a near impossible task, testing of newly developed feature by mimicking the production like scenario.

We look to automate this scenario by adopting DevOps to provision local Environment on PC, Mac or Linux to mimic production like deployments, and help the developer test their code thoroughly before submitting a pull request, hence maintaining the sanctity of our repository code and also eliminating any issues post actual deployments.

Below are the possible ways to achieve this goal, though may not be mutually exclusive approaches, but will help us sum up the discussion,

*   **1.)** Isolating development environment: Using Virtualization tools like Vagrant on top of Vmware or Virtual box to isolate development environment, and run Airavata on separate virtual OS.

*  
*   **2.)** Containerization of code base: Docker is the answer here, a big undertaking for initial development, but makes the life easy after it is achieved. To top it off, there are docker containers already written for Airavata, which will make the development faster.

*  
*   **3.)** Installing directly on the local machine: Write scripts to run Airavata directly on host operating systems, but this approach is not feasible, as support for different OS's is required also provisioning on Windows and Linux will follow completely different approaches.

The solution we have choosen is actually a combination of first two approaches, we need isolation of environment but that will be achieved by using Docker containers, which instead will be provisioned by vagrant and Ansible. The main goal here is to write a generic Ansible script to deploy application in all aspects SD lifecycle, be it Local or on cloud, the same docker conatainer is deployed with the help of Ansible. Vagrant is choosen just to create abstraction layer to provide OS agnostic enviroment for our web application.

A detailed comparison of all the all the infrastructure-as-Code is written in our Class repositories wiki page, refer this [blog](blogs/DevOps-tool-Comparison)

### CI & CD post-GitHub pull requests

![jenkins-CICD](https://www.ravellosystems.com/sites/all/themes/ravello/images/new-images/customer-case-studies/deutsche-telekom/dt-cicd.jpg)

This topic is core to our DevOps efforts, I would definitely recommend everyone reading this post to go through a blog written by Atlassian Team explaining core concepts of CI/CD https://www.atlassian.com/blog/continuous-delivery/practical-continuous-deployment

Presently, Airavata's GitHub repo has travis CI builds enabled, that is, whenever a pull request is submitted by the developer, Travis builds and tests the whole Airavata project using Maven, which is great, because we want to check whether the pull requests for any inefficiencies and build failures, before we merge it to our develop branch.

Travis CI builds provide just a partial implementation of CI/CD pipeline, major part of implementing the DevOps lifecycle is to design the complete pipeline with continuous deployment, though Travis CI provides integrations to cloud providers like AWS, but it lacks support for in-house infrastructure and also OpenStack, hence makes no sense to invest time in Travis CI. Therefore, we have choosen Jenkins for accomplishing the CI & CD implementations, especially because of the fine grained controls it provides through Jenkins pipelines and the support for continuous deployments through Jenkins slaves. While making this decision, we considered many different tools, like Atlassian's Bamboo, Netflix's spinnaker etc. Though they feel promising, they lack the wide community support which Jenkins has.

**Note:** A complete setup using jenkins for CI/CD blog with implementation details will posted shortly.

### Upgrading application on Dev, Staging and Production Environment

This plays a crucial part in continuous deployment pipelines, at Apache Airavata, the middleware is composed of many micro-services, hence a very fault tolerant, high available and a robust infrastructure is needed. One of the main reason why the continuous deployment pipelines were not implemented at Airavata is because of these reasons, it is an ardous task to occomplish. Now, lets discuss the available approaches,

  * **-** **Blue-Green Deployment:** One of the challenges with automating deployment is the cut-over itself, taking software from the final stage of testing to live production. You usually need to do this quickly in order to minimize downtime. The blue-green deployment approach does this by ensuring you have two production environments, as identical as possible. At any time one of them, let's say blue for the example, is live. As you prepare a new release of your software you do your final stage of testing in the green environment. Once the software is working in the green environment, you switch the router so that all incoming requests go to the green environment - the blue one is now idle. Blue-green deployment also gives you a rapid way to rollback - if anything goes wrong you switch the router back to your blue environment.

  ![blu-green deployments](https://www.martinfowler.com/bliki/images/blueGreenDeployment/blue_green_deployments.png)

  * **-** **In-place Deployements:** In-place deployments are nothing but upgrading the current revision of the system with newer version, A more traditional approach to deploy application, easy to develop and uses less infrastructure provisioning, but the fault tolerance is hard to achieve, mainly because the roll-backs are difficult in this scenario. But the deployements are fast, taking care of high availablility aspect.

  ![in-place deployments](https://image.slidesharecdn.com/continuousdeliveryanddeploymentonaws-150114202455-conversion-gate02/95/continuous-delivery-and-deployment-on-aws-25-638.jpg?cb=1443566178)

### Debugging deployed code base

This particular feature is an interesting problem in itself, majority of the Apache Airavata Code base in Java, therefore debugging in Java is mainly carried out using the Java Platform Debugger Architecture (JPDA) framework, in simple terms debugging in run time can be achieved by placing stratergic break points in source code, which are interpretted by JVM with the help of Java Debug Wire Protocol (JDWP) and Java Debug Interface (JDI), essentially stopping the execution when a break point is reached. For more details please refer [Java Doc](http://docs.oracle.com/javase/6/docs/technotes/guides/jpda/) .

Debugging is easy when carried out in local environment, but the main challenge is to achieve this for deployed code, right now according my knowledge only few IDE's provide this feature, [IntelliJ Idea  IDE](https://www.jetbrains.com/idea/) refer the [documentation](https://www.jetbrains.com/help/idea/2016.3/remote-debugging.html) for remote debug feature  

## __Conclusion__

The solutions we are looking for are already listed in the above sections, here is a summary:

  * **-** **Local provisioning for Development & Testing:** Isolation of environment achieved by Docker containers, which instead will be provisioned by vagrant and Ansible. A generic Ansible script to deploy application in all aspects SD lifecycle, be it Local or on cloud, the same docker conatainer is deployed with the help of Ansible and Vagrant as OS abstraction layer.
  * **-** **CI & CD post-GitHub pull requests:** Jenkins pipelines, Continuous Integration test with the help of Maven and Continuous deployments using Jenkins slaves.
  * **-** **Upgrading application on Dev, Staging and Production Environment:** Since Apache Airavta is composed of many microservices, both Blue-green and In-place are suited for different aspects of the systems.
  * **-** **Debugging deployed code base:** Remote debugging provided by Intellij Idea IDE.

## __My Git Commits for this Project__

Anuj Bhandar's Commit History : https://github.com/airavata-courses/spring17-devops/commits?author=anujbhan

## __My Discussions on the Apache Airavata Developer List__

Anuj Bhandar's Dev list activity:
* http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3C1a9ac001-7055-e4a4-dc62-0661d3510b6d%40gmail.com%3E
* http://mail-archives.apache.org/mod_mbox/airavata-dev/201702.mbox/%3C7ce261bc-368e-efa7-8bdb-ff2be823a125%40gmail.com%3E
