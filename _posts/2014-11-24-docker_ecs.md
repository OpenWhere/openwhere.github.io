---
layout: post
title:  "Thoughts on Elastic Container Service"
date:   2014-11-23 18:30:04
author: aaslinger
categories: docker amazon AWS devops continuous delivery
---

##Thoughts on Docker
Docker is a very exciting technology and something that we at OpenWhere have recently been embracing and leveraging to much success. Besides isolation, one of the largest and most overlooked benefits of Docker is that it provides a unit of work pattern and interface (start / stop / volumes / linking etc.) that works across environments. Docker containers don't have to contain long running Virtual Machine type images, but can also support atomic units of work. For example, if you wanted to convert and project an image to a different format without needing all the specially compiled C libraries installed on your machine. Before Docker, tasks like these were also DevOps nightmares as the libraries may build differently and have different dependency issues in different environments.

However, currently most of the benefit of the Docker ecosystem is centralized around that container or multiple containers working together on a host. There are some gaps in a cloud based environment where services may be spread across many hosts, or one container per instance since instances may be sized small. The two largest of these gaps are:

 - Service Discovery 
 - Orchestration
 
There are quite a few approaches to service discovery gaining traction which I will talk about in a future blog post. Elastic container Service (ECS) looks to largely address orchestration in the AWS cloud. It should also be noted that those interested in using Mesos should also consider a potential Mesos [alternative](http://mesos.apache.org/documentation/latest/docker-containerizer/) and google also provides a [container engine](https://cloud.google.com/compute/docs/containers).

##Elastic Container Service
Elastic Container Service (ECS) was introduced during the second keynote of the 2014 re:Invent conference. Of the keynote, 3 of the sessions had a Docker focus which was the most time devoted to any topic. In my experience in the general sessions, Docker was also the hottest topic which was pretty easy to measure just be the long lines and people "camped out" to get into the sessions. The first session of the Elastic Container Service had a line that was so long it left the conference facility and most people did not get in. There was also a very long line for the encore session on the final day of the conference when many people were also not there. This just shows the level of excitement around Docker and the potential for Elastic Container Service.

Amazon stated that they had several goals while designing the container service based on customer feedback. One goal from the session was "Docker + cloud = AWSOME". By this they mean making Docker super easy and enjoyable to use on AWS. One of the most important goals I believe is making Containers a first class citizen (Like EC2 or any other top level service). In order for a container based process to remain manageable, you really only want to deal with containers and minimize any work with the host infrastructure. Other goals included the following:

- containers @ every scale
- optimized scheduling
- consideration of resource requirements
- isolation requirements
- improve resource efficiency (of cluster running containers)
- simple API to integrate with
- possible to extend the scheduling (such as with Mesos) 

Also important is that ECS respects all the core principals of Docker so you don't have to fundamentally change your approach to an "AWS" way. Linked containers, tying containers part of a work product together, Docker hub integration, etc. are all supported.

By providing Docker container scheduling and orchestration, these are some of the problems that ECS helps resolve:

- **Cluster Management**: It can be painful to manage the host infrastructure for your Docker containers. Especially if you want to maximize resource utilization.
- **Scale**: It should be easy to scale your cluster with your containers
- **Configuration Management**: Docker provides great configuration management of individual containers but there are gaps when you look to manage multiple containers that must be wired together to perform a task. Without ECS you need to use something external to capture and version control this wiring.
- **Container Sprawl**: Is a potential problem. Tooling can help mitigate and manage your containers. I've already found with continuous deployment you need special code to keep your hosts clean.
- **Availability**: Docker isn't going to do anything for you here out of the box. Its nice if your orchestration layer can help.
- **Security**: The Docker security story is still in work as it came out of its 1.0 release and just got better with signed containers in 1.3. IAM Roles and other integrations for AWS are a useful addition in the cloud.


##Concepts
Docker has two primary concepts images and containers. Containers are instances of images on some host. ECS tweaks this a bit with some new concepts which are specified in JSON.

- **Tasks**: Are units of work and may include multiple containers. This concept will be very useful in capturing what could be messy linkages among containers for a specific task.
- **Containers**: Comprise the definition of the container, with all the parameters you would use with `docker run`. It will be nice to have this abstraction documented in version control and ECS still allows you to override.
- **Clusters**: Are a pool of resources for tasks
- **Container Instances**: is an instance which takes tasks that are scheduled

##Future
It looks like AWS made a push to get ECS out in time for re:Invent (In limited preview). There are still some future integrations and potential gaps they are working on. The biggest of these for our team is CloudFormation as we use this heavily to automate all our environments. Some of these gaps are:

- ELB Support
- Cloudwatch
- Cloud Watch Logs
- CloudFormation
- Tags
- AWS Management Console
- Partner AMIs
- Building a UI

One of the tradeoffs people will have to make is how much tooling or process do you build now to get around the gaps. It can be dangerous to do so as AWS is sure to release more and more functionality over time as this year's re:Invent was evidence of.

##Summary
ECS looks like an extremely promising tool to use with Docker. We are likely to fully adopt the technology as it provides one of the last gaps for us, orchestration, and should minimize time spent on infrastructure and deployment issues. One of the other benefits that I am anticipating is that it should help institutionalize Docker best practices and make things like running a single process per Docker container easier, as it will now be more easier to control groupings of containers themselves. Stay tuned as we report back in the future on ECS lessons learned. 