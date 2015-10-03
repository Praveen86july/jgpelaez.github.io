---
layout: post
title: Provisioning cloud infrastructure with Terraform, Jenkins and docker. Part I
description	: Basic setup
date: 04/10/2015
author: "Juan Carlos G. Peláez"
comments: true
header-img: "media/background/gibraltar-1.jpg"
tags: [terraform, aws, docker, jenkins, ci, continous integration, cloud]
tagsEn: [terraform, aws, docker, jenkins, ci, continous integration, cloud]
---

In the world of development the continuous integration has become a necessary tool in our lives. Is important that everything uploaded to the version control system by the developers compiles, and it's test runs properly. Also the	 same system is used for all the stages till arriving to production.

One of the most famous tools is Jenkins (very well known in the Java community).. 

The question is, can we use this same tools to check if our infrastructure as code (using for example terraform) can be continuously checked, and also used to populate the changes in the real enviromnents?

In this series of articles, I will try to demonstrate that is possible and also very useful at it is for the software deveploment. 

# Installation

First we need to install our tools in a server, it could be done in local, but it will be more useful to make it accessible to other member of the project. So, we are going to create a VM it in the cloud, and the install the software.

For this installation we are going also to use terraform. We create an initial folder in our project
jenkins-service , with the files:

## terraform
 
### main.tf

We create a security group with port 22 only open to our ip, and the same for the port 80. As we are not going jet to create security in our jenkins, and is not used throught ssl is better not to open it to the world.

The EC2 instance will be created with the linux AWS ami, as it has installed by default docker and we trust AWS in the creation of an AMI.

```json
the file contet

``` 
### variables.tf

## Provision of jenkins

The ec2 instance will be provision with jenkins, and we are going to use the Docker image from the docker hub.

As it's necesary some extra instalations, and also make it posible to run a docker container inside the jenkins container.

We got some extra information for this creation from other blog posts:

http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/
TODO, find the other instalation.

In our Dockerfile we add some extra plugins to be userd

https://wiki.jenkins-ci.org/display/JENKINS/Post+build+task

```Dockerfile
the file content

```

Containing our variables

And finally we run the shell script running the jenkins container:

## Execution

We can obtain the sample code from github:
```sh
git clone https://github.com/jgpelaez/terraform-aws-jenkins-docker-sample.git
cd 
```

```sh
docker run \
	--rm \
	-v $(pwd):/data \
	-v /home/jgpelaez/git/ec2-instance/data/:/data-ssl \
    uzyexe/terraform plan \
    -var 'key_name=keypar2' -var 'key_path=/data-ssl/keypar2.pem'
```

## Jenkins configuration

First we are going to check if a plan runing a docker hello world can be executed.

Create a New Item

Freestyle Project

in the Build, we select execute shell, and 
docker run hello-world

After buildng the job, will get the message:

Started by user anonymous
Building in workspace /var/jenkins_home/jobs/tmp1/workspace
[workspace] $ /bin/sh -xe /tmp/hudson8602353761148029617.sh
+ docker run hello-world
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (Assuming it was not already locally available.)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 http://docs.docker.com/userguide/
Finished: SUCCESS


## Checking terraform plan

In this first article we create a jenkins job, that will check for every change commited in the version control system, there are not errors. 
It will be a "compilation" check plan, and for the moment will not create any infrasctucture.

for the execution of terraform is also used docker. 
The execution of terraform plan will give a result, in case of error, will configure a notification, using one of the notifications plugins available.

# Conclusion


## Resources

- Source code in [github/jgpelaez] (https://github.com/jgpelaez/terraform-aws-wso2-esb-docker-sample).

## Thanks / Contributions

-  If it's useful code you can favourite it in:
  [GitHub] (https://github.com/jgpelaez/terraform-aws-wso2-esb-docker-sample/stargazers)
