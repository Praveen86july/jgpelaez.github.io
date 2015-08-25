---
layout: post
title: Infrastucture as code with Terraform and docker
description	: Example of use of Terraform with docker, WSO2 ESB in AWS 
date: 27/08/2015 12:00:00
author: "Juan Carlos G. Pelaez"
comments: true
tags: [terraform, aws, docker, wso2 esb, esb]
---

One of the good practices for a datacenter or a cloud infrastructure is to have the infrastructure as code
(http://www.thoughtworks.com/es/insights/blog/infrastructure-code-reason-smile).
There are many different options in the marked to create and provision the insfrastucture and code. 
In this article it's going to be explained how to create a simple infrastructure in [AWS](https://aws.amazon.com/).
The sample application will have an (Elastic Load Balancer)[https://aws.amazon.com/elasticloadbalancing/], an EC2 instance with a WSO2 ESB, and a axis server with a sample service.

The sample can be executed in the (AWS free tier)[http://aws.amazon.com/free/] without cost.

First it's necessary to create an account in AWS or use an existing one.


[Create secret key](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html)

![Create secret key] (/media/posts/terraform-aws-wso2-esb-docker-sample/aws-iam-create-access-key.png)

EC2 dashboard, Network & security
![Create key pair] (/media/posts/terraform-aws-wso2-esb-docker-sample/aws-create-key-pair.png)

Use the docker image

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform plan 

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform apply

not necesary to install terraform in the computer, can use the docker image.

Destroy:

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform plan -destroy

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform destroy -force 


![logs] (/media/posts/terraform-aws-wso2-esb-docker-sample/wso2-esb-console.png)

```sh
sh run-client.sh terraform-example-elb-1103743156.eu-west-1.elb.amazonaws.com
```
![logs] (/media/posts/terraform-aws-wso2-esb-docker-sample/wso2-esb-logs.png)