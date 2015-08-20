---
layout: post
title: Using Docker registration for WSO2 images
subtitle: ""
date: 11/07/2015 12:00:00
author: "Juan Carlos G. Pelaez"
comments: true
---

## Introduction

Once known as [create a picture of Docker] (https://itscenario.wordpress.com/2014/11/09/dockerizing-wso2-esb/), if we want to share this with the team, or use it to different environments, it is possible to export to a tar file, or you can use the file [Dockerfile] (https://docs.docker.com/reference/builder/) to build back the image on each of the docker host where it will be used.
Having to rebuild the image on the pc of each of the components of your computer is not practical. Would not it be better if we could download images and ready to use? To resolve this problem have an element docker called registration, which can raise and lower our images.

One of the great advantages that allow us to check is to be able to test different versions of a product with the sole modification of the image to download if they are on record.

We can install a private registry or use the [public registry available in Docker] (https://registry.hub.docker.com/). This log can be used by any user, and can be searched to see if another user has added the image we want.

! [Registration docker] (/ media / wso2-in-docker-registry / search-docker-registry.png)

If you will use a file uploaded by another user registration is recommended to use the marked "automatic building". In these images we can see how they have been created with the [Dockerfile] (https://docs.docker.com/reference/builder/) if they have been uploaded directly by a user should not rely on it if it is not a user confidence, because it is an image that can contain malicious code.

For use in production of Docker's write our own images as the images of the registry can be deleted by the user who uploaded. We also recommend using images marked as official, these have been developed either by the vendor, by the team of Docker, or by users with much popularity.

! [Registration docker] (/ media / wso2-in-docker-registry / docker-official-repo.png)

In this post we will explain how to upload your images WSO2 registration.

## Creating images in the public register of Docker

First we create a public repository on github or bitbucket or we will [fork existing one] (https://github.com/jgpelaez/docker-wso2-esb.git).

In this repository add the / s [Dockerfile] (https://docs.docker.com/reference/builder/):

`` `Docker
FROM java: openjdk-7

MAINTAINER juancarlosgpelaez@gmail.com

ENV WSO2_URL = https: //s3-us-west-2.amazonaws.com/wso2-stratos
ENV WSO2_SOFT_VER = 4.8.1
RUN \
mkdir -p / opt && \
wget -P / opt $ WSO2_URL / wso2esb- $ WSO2_SOFT_VER.zip && \
    /opt/wso2esb-$WSO2_SOFT_VER.zip unzip -d / opt && \
    rm /opt/wso2esb-$WSO2_SOFT_VER.zip

ESB https port #
EXPOSE 9443
# Http ESB transport pass-through port
EXPOSE 8280
# ESB https transport pass-through port
EXPOSE 8243

ENV JAVA_HOME = / usr
CMD ["/opt/wso2esb-4.8.1/bin/wso2server.sh"]
`` `

Create a user in the [registry docker] (https://registry.hub.docker.com/)

! [Registration docker] (/ media / wso2-in-docker-registry / docker-registry-signup.png)


We can use our GitHub user for registration.

After creating the account can create a new repository, automatic construction, the registry read our [Dockerfile] (https://docs.docker.com/reference/builder/) and automatically perform construction:

! [Registration docker] (/ media / wso2-in-docker-registry / docker-registry-repositories.png)

We chose our bitbucket account or github:

! [Registration docker] (/ media / wso2-in-docker-registry / docker-registry-git-source.png)

And select the repository that is the / the [Dockerfile] (https://docs.docker.com/reference/builder/)/s:

! [Registration docker] (/ media / wso2-in-docker-registry / docker-registry-git-source-repository.png)

We can selecting the directory where you will find the [Dockerfile] (https://docs.docker.com/reference/builder/). In our case we will add several [Dockerfile] (https://docs.docker.com/reference/builder/) for different versions of the WSO2 ESB.
We can also change the name of the repository, the convention is to use the git repository docker- [vendor] - [software] for recording docker [vendor] - [software].

! [Creating repository] (/ media / wso2-in-docker-registry / docker-registry-repository-creation.png)

The construction can be run manually, but usually will not be necessary, because when we change a file in our depository for git, construction runs automatically.

! [Registration docker] (/ media / wso2-in-docker-registry / docker-registry-repository-build-details.png)

## Using the public registry of Docker

Once the images have created or want to try a new image uploaded by another user we can do with the run command, for example:

`` `
docker run -p 19443: 9443 jgpelaez / wso2-ESB
`` `

So we will be running an instance indicating the container, the container port 9443 will be exposed to the host on port 19443 and the tag used will'latest'. To see more execution parameters we can see with the help'docker --help' run.

In this case we see the management console with the URL in your browser:

`` `
https: // localhost: 19443 / carbon / admin / login.jsp
`` `

! [Registration docker] (/ media / wso2-in-docker-registry / docker-wso2-ESB-admin.png)

The host can enter in the url it depends on the system in which we execute docker. If you natively docker (ubuntu, redhat, etc) will be localhost, if we use boot2docker (Windows / Mac), will be the IP of the virtual machine created by [boot2docker] (http://boot2docker.io/) (by defect usually 192.168.59.103).

Among other options we could try examples of wso2 by instruction:

`` `
docker run -p 19443: 9443 jgpelaez / wso2-ESB /opt/wso2esb-4.8.1/bin/wso2esb-samples.sh -sn [sample number]
`` `

A great advantage is that it provides us docker can run different instances of the same container port host changing, always using one free, or with the 'P' to automatically assign a free port. We see the executions of containers with instruction

`` `
ps docker
`` `

`` `
ID CONTAINER PORTS STATUS IMAGE CREATED COMMAND NAMES
dbfa29da863f jgpelaez / wso2-esb "/opt/wso2esb-4.8.1/ 44 minutes ago 44 minutes Up 8243 / tcp, 8280 / tcp, 0.0.0.0:9443->9443/tcp happy_goodall
`` `

## Conclusion

The use of Linux containers (in this case Docker) and an image recording can help us a lot to get isolated and repeatable environments.
Docker registration or a private registry will help us in developing for all team members to work with the same configuration.
From a DevOps perspective, if we use images of Docker for development, test and production records, we testeando in a very similar environment in all phases of project / product, and minizaremos configuration errors that could reach production.

## Resources

- Source from [github / jgpelaez] (https://github.com/jgpelaez/docker-wso2-esb.git).
- Image of the WSO2 ESB [repository docker] (https://registry.hub.docker.com/u/jgpelaez/wso2-esb/)

## Thanks / Contributions

- You can contribute to docker record: https://github.com/jgpelaez/docker-wso2-esb creating "pull requests"
- If it seems useful code you can bookmark it in:
  [Github] (https://github.com/jgpelaez/docker-wso2-esb/stargazers)
  [Docker Registry] (https://registry.hub.docker.com/u/jgpelaez/wso2-esb/)