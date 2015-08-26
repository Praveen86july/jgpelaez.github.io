---
layout: post
title: Infrastucture as code with Terraform and docker
description	: Example of use of Terraform with docker, WSO2 ESB in AWS 
date: 27/08/2015 12:00:00
author: "Juan Carlos G. Peláez"
comments: true
header-img: "media/background/plane-clouds.jpg"
tags: [terraform, aws, docker, wso2 esb, esb]
---

One of the good practices for a datacenter or a cloud infrastructure is to have the [infrastructure as code]
(http://www.thoughtworks.com/es/insights/blog/infrastructure-code-reason-smile).
There are many different options in the market as CloudFormation, Heat, etc to create and provision the insfrastucture and code. 
On internet we can find different comparisons for the tools:

- [thoughtworks.com](http://www.thoughtworks.com/es/insights/blog/choosing-right-tool-provision-aws-infrastructure)
- [ypoonawala.wordpress.com](https://ypoonawala.wordpress.com/2014/12/14/terraform-vs-cloudformation-a-comparison-of-infra-management-tools/)
- [www.terraform.io](https://www.terraform.io/intro/vs/index.html)
 
In this article it's going to be explained how to create a simple infrastructure in [AWS](https://aws.amazon.com/).
The sample application will have an [Elastic Load Balancer](https://aws.amazon.com/elasticloadbalancing/), an EC2 instance with a WSO2 ESB, and a axis server with a sample service. This sample is based in the [BASIC TWO-TIER AWS ARCHITECTURE](https://www.terraform.io/intro/examples/aws.html) from terraform samples.

# AWS account

The sample can be executed in the [AWS free tier](http://aws.amazon.com/free/) without cost.

First it's necessary to create an account in AWS or use an existing one.

Once we have our AWS account, it's necesary to **[create a secret key](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html)** on aws.

![Create secret key] (/media/posts/terraform-aws-wso2-esb-docker-sample/aws-iam-create-access-key.png)

This secret key and password will allow to access AWS with the APIs.

# Use of terraform

Terraform can be installed [following the instructions](https://www.terraform.io/intro/getting-started/install.html), but in this case we can avoid it using a docker image doing a search in the **[Docker Hub](https://hub.docker.com/)**. Using the docker image it's not needed to install additional software in your computer, and it also allows to use terraform in unsupported operating systems. In a team everybody will use the same version of terraform if we specify it in the execution scripts.

To obtain the docker image, we can pull it, in this case:

```sh
docker pull uzyexe/terraform
```

## Terraform project

In this moment we can create the terraform files (all the tf files will be executed in parallel if there is no dependency between instructions):

### **outputs.tf**: 

Definition for the output to show in the screen

```json
output "ebs_address" {
  value = "${aws_elb.web.dns_name}"
}

output "ec_address" {
  value = "${aws_instance.web.dns_name}"
}

```

### **variables.tf**: 
Definition for some input variables, the keys, and passwords will be parametrized

```json
variable "key_name" {
    description = "Name of the SSH keypair to use in AWS."
}

variable "key_path" {
    description = "Path to the private portion of the SSH key specified."
}

variable "access_key" {}
variable "secret_key" {}

variable "aws_region" {
    description = "AWS region to launch servers."
    default = "eu-west-1"
}

# Ubuntu Server 14.04 LTS (x64)
variable "aws_amis" {
    default = {
        eu-west-1 = "ami-5da23a2a"
    }
}

```

### **terraform.tfvars**: 
Replacement for the variables, in this file we put the user data information, for example the user key and password. This file is ignored in the public git repository. We can store this files in private repositories (in bitbucket we can have **[unlimited private repositories for free] (https://bitbucket.org/plans)**)

### **main.tf**: 

Main file with our code for the creation.

In this file it's created an AWS security group, [Elastic Load Balancer] (https://aws.amazon.com/elasticloadbalancing/) and an AWS instance.

[Elastic Load Balance Pattern](http://www.cloudcomputingpatterns.org/Elastic_Load_Balancer):
![Elastic Load Balance Pattern] (http://www.cloudcomputingpatterns.org/images/e/ef/Elastic_load_balancer_sketch.png)
 


With the [Elastic Load Balancer] (https://aws.amazon.com/elasticloadbalancing/) we would have in the future the possibility to dinamically add more services to our infrastructure. 
The instance is provisioned with simple shell scripts installing Docker images. The machine can be also provisioned with a [custom Amazon Machine Image (AMI)](https://aws.amazon.com/marketplace/help/200940360) or with a provisioner like **[Packer](https://www.packer.io/)**, **[Ansible](http://www.ansible.com/home)**, **[Chef](https://www.chef.io/)**, **[Puppet](https://puppetlabs.com/)**... 

```json
# Specify the provider and access details
provider "aws" {
    access_key = "${var.access_key}"
    secret_key = "${var.secret_key}"
    region = "${var.aws_region}"
}

# Our default security group to access
# the instances over SSH and HTTP
resource "aws_security_group" "default" {
    name = "terraform_example"
    description = "Used in the terraform"

    # SSH access from anywhere
    ingress {
        from_port = 22
        to_port = 22
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    # HTTP access from anywhere
    ingress {
        from_port = 443
        to_port = 443
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
    
    # HTTP access from anywhere
    ingress {
        from_port = 80
        to_port = 80
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }

    # outbound internet access
    egress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }
}


resource "aws_elb" "web" {
  name = "terraform-example-elb"

  # The same availability zone as our instance
  availability_zones = ["${aws_instance.web.availability_zone}"]

  listener {
    instance_port = 80
    instance_protocol = "http"
    lb_port = 80
    lb_protocol = "http"
  }
  
  # The instance is registered automatically
  instances = ["${aws_instance.web.id}"]
}

resource "aws_instance" "web" {
  # The connection block tells our provisioner how to
  # communicate with the resource (instance)
  connection {
    # The default username for our AMI
    user = "ubuntu"

    # The path to your keyfile
    key_file = "${var.key_path}"
  }

  instance_type = "t1.micro"

  # Lookup the correct AMI based on the region
  # we specified
  ami = "${lookup(var.aws_amis, var.aws_region)}"

  # The name of our SSH keypair you've created and downloaded
  # from the AWS console.
  #
  # https://console.aws.amazon.com/ec2/v2/home?region=us-west-2#KeyPairs:
  #
  key_name = "${var.key_name}"

  # Our Security group to allow HTTP and SSH access
  security_groups = ["${aws_security_group.default.name}"]

  # We run a remote provisioner on the instance after creating it.
  provisioner "file" {
      source = "./wso2-esb/"
      destination = "/tmp/"
  }	
provisioner "remote-exec" {
   inline = [
        "sh /tmp/install-docker.sh",
        "sh /tmp/download-docker-images.sh",        
        "sh /tmp/run-axis-server.sh",
        "sh /tmp/run-esb.sh"
    ]
  }
}
```
The shell scripts for provision the machine 


[Stateless Pattern] (http://www.cloudcomputingpatterns.org/Stateless_Component):

![Stateless Pattern] (http://www.cloudcomputingpatterns.org/images/2/29/Stateless_component_sketch.png)



## Terraform execution


EC2 dashboard, Network & security
![Create key pair](/media/posts/terraform-aws-wso2-esb-docker-sample/aws-create-key-pair.png)

Use the docker image

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform plan 

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform apply

not necesary to install terraform in the computer, can use the docker image.

Destroy:

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform plan -destroy

docker run --rm -v $(pwd):/data --net=host jgpelaez/terraform destroy -force 

![terraform-apply-finised.png] (/media/posts/terraform-aws-wso2-esb-docker-sample/terraform-apply-finised.png)

![aws-instances.png] (/media/posts/terraform-aws-wso2-esb-docker-sample/aws-instances.png)

![aws-instances.png] (/media/posts/terraform-aws-wso2-esb-docker-sample/aws-instance-detail.png)

![aws-security-groups.png] (/media/posts/terraform-aws-wso2-esb-docker-sample/aws-security-groups.png)

![aws-elb.png] (/media/posts/terraform-aws-wso2-esb-docker-sample/aws-elb.png)

![logs] (/media/posts/terraform-aws-wso2-esb-docker-sample/wso2-esb-console.png)


![logs] (/media/posts/terraform-aws-wso2-esb-docker-sample/wso2-esb-logs.png)

## Resources

- Source code in [github/jgpelaez] (https://github.com/jgpelaez/terraform-aws-wso2-esb-docker-sample).

## Thanks / Contributions

-  If it's useful code you can favourite it in:
  [GitHub] (https://github.com/jgpelaez/terraform-aws-wso2-esb-docker-sample/stargazers)
