---
layout:     post
title: Using Docker registry for WSO2 images
subtitle:   ""
date:       2015-07-11 12:00:00
author:     "Juan Carlos G. Peláez"
comments: true
---

## Introduction

Once we know [how to create a Docker image] (https://itscenario.wordpress.com/2014/11/09/dockerizing-wso2-esb/), if we want to share this image with the team, or use it in different environments, it´s possible to export to a tar file, or you can use the [Dockerfile] (https://docs.docker.com/reference/builder/) to build the image on each of the docker hosts where it will be used.
Having to rebuild the image on the pc of each of the components of your team is not practical. Would not it be better if we could download images ready to use? To resolve this problem Docker has a solution called registry, which can raise and download our images.

One of the great advantages is to be able to test different versions of a product only changing the version of the image to download.

We can install a private registry or use the [public registry available in Docker] (https://registry.hub.docker.com/). This registry can be used by any user, and has a search function to see if another user has added the image we want.

! [Registration docker] (/ media / wso2-in-docker-registry / search-docker-registry.png)

If you use an image uploaded by another user registration is recommended to use the ones marked as "automatic building". In these images we can see how it has been created with the [Dockerfile] (https://docs.docker.com/reference/builder/) if the image has been uploaded directly by a user whe should'nt trust its, because it is an image that can contain malicious software.

TODO

For use in production of Docker's write our own pictures as the images of the registry can be deleted by the user who uploaded. We also recommend using images marked as official, these have been developed either by the seller, by the team of Docker, or by users with much popularity.

! [Registration docker] (/ media / wso2-in-docker-registry / docker-official-repo.png)

In this post we will explain how to upload images of WSO2 nuestrar registration.

## Creación de imágenes en el registro público de Docker

Primero crearemos un repositorio público en github o bitbucket, o bien haremos un [fork de uno existente] (https://github.com/jgpelaez/docker-wso2-esb.git).

A este repositorio añadiremos el fichero/s [Dockerfile] (https://docs.docker.com/reference/builder/):

```docker
FROM java:openjdk-7

MAINTAINER juancarlosgpelaez@gmail.com

ENV WSO2_URL=https://s3-us-west-2.amazonaws.com/wso2-stratos
ENV WSO2_SOFT_VER=4.8.1
RUN  \
	mkdir -p /opt && \
	wget -P /opt $WSO2_URL/wso2esb-$WSO2_SOFT_VER.zip && \
    unzip /opt/wso2esb-$WSO2_SOFT_VER.zip -d /opt && \
    rm /opt/wso2esb-$WSO2_SOFT_VER.zip

# ESB https port
EXPOSE 9443
# ESB http pass-through transport port
EXPOSE 8280
# ESB https pass-through transport port
EXPOSE 8243

ENV JAVA_HOME=/usr
CMD ["/opt/wso2esb-4.8.1/bin/wso2server.sh"]
```

Crearemos un usuario en el [registro de docker] (https://registry.hub.docker.com/)

![registro en docker](/media/wso2-in-docker-registry/docker-registry-signup.png)


Podemos utilizar nuestro usuario de GitHub para realizar el registro.

Una vez creada la cuenta podremos crear un nuevo repositorio, con construcción automática, el registro leerá nuestro [Dockerfile] (https://docs.docker.com/reference/builder/) y automáticamente realizará la construcción:

![registro en docker](/media/wso2-in-docker-registry/docker-registry-repositories.png)

Elegimos nuestra cuenta de bitbucket o de github:

![registro en docker](/media/wso2-in-docker-registry/docker-registry-git-source.png)

Y seleccionamos el repositorio en el que se encuentra el/los [Dockerfile] (https://docs.docker.com/reference/builder/)/s:

![registro en docker](/media/wso2-in-docker-registry/docker-registry-git-source-repository.png)

Podremos selecionar el directorio en el que se encuentra el [Dockerfile] (https://docs.docker.com/reference/builder/). En nuestro caso añadiremos varios [Dockerfile] (https://docs.docker.com/reference/builder/) para diferentes versiones del ESB de WSO2. 
También podremos cambiar el nombre del repositorio, la convención que utilizamos es para el repositorio en git docker-[vendedor]-[software] y para el registro de docker [vendedor]-[software].

![Creación del repositorio](/media/wso2-in-docker-registry/docker-registry-repository-creation.png)

La construcción se puede ejecutar manualmente, aunque normalmente no será necesario, ya que cuando modificamos un fichero en nuestro repositoria de git, se ejecuta automáticamente la construcción.

![registro en docker](/media/wso2-in-docker-registry/docker-registry-repository-build-details.png)

## Utilizando el registro público de Docker

Una vez tenemos las imágenes creadas o bien queremos probar una nueva imagen subida por otro usuario podremos hacerlo con la instrucción run, por ejemplo:

```
docker run -p 19443:9443 jgpelaez/wso2-esb
```

Así estaremos indicando que ejecute una instancia del contenedor, el puerto del contenedor 9443 será expuesto al host por el puerto 19443 y la etiqueta utilizada será ´latest´. Para poder ver más parámetros de ejecución podemos ver la ayuda con ´docker run --help´.

En este caso podremos ver la consola de administración con la url en el navegador:

```
https://localhost:19443/carbon/admin/login.jsp
```

![registro en docker](/media/wso2-in-docker-registry/docker-wso2-esb-admin.png)

El host a indicar en la url dependerá del sistema en el que ejecutemos docker. Si tiene de forma nativa docker (ubuntu, redhat, etc), será localhost, si utilizamos boot2docker (windows/mac), será la ip de la máquina virtual creada por [boot2docker] (http://boot2docker.io/) (por defecto suele ser 192.168.59.103).

Entre muchas otras opciones podriamos probar los ejemplos del wso2 mediante la instrucción:

```
docker run -p 19443:9443 jgpelaez/wso2-esb /opt/wso2esb-4.8.1/bin/wso2esb-samples.sh -sn [sample number]
```

Una gran ventaja que nos proporciona docker es que podemos ejecutar diferentes instancias del mismo contenedor cambiando el puerto del host, utilizando siempre uno libre, o con la opcion '-P' que asignará automáticamente un puerto libre. Podremos ver las ejecuciones de los contenedores con la instrucción 

```
docker ps
```

```
CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS              PORTS                                        NAMES
dbfa29da863f        jgpelaez/wso2-esb   "/opt/wso2esb-4.8.1/   44 minutes ago      Up 44 minutes       8243/tcp, 8280/tcp, 0.0.0.0:9443->9443/tcp   happy_goodall      
```	

## Conclusión

El uso de contenedores linux (en este caso Docker) y de un registro de imágenes nos puede ayudar mucho para conseguir entornos aislados y repetibles. 
El registro de Docker o un registro privado nos podrá ayudar para que en el desarrollo todos los miembros del equipo trabajen con la misma configuración.
Desde una perspectiva DevOps, si utilizamos imágenes de Docker para desarrollo, test y producción con un registro, estaremos testeando en un entorno muy similar en todas las fases del proyecto/producto, y minizaremos los errores de configuración que puedan llegar a producción.

## Recursos

- Código fuente en [github/jgpelaez] (https://github.com/jgpelaez/docker-wso2-esb.git).
-  Imágen del wso2 esb en [repositorio docker] (https://registry.hub.docker.com/u/jgpelaez/wso2-esb/) 

## Gracias / Contribuciones

- Puedes contribuir al registro docker: https://github.com/jgpelaez/docker-wso2-esb creando "pull requests"
- Si parece útil el código puedes márcarlo como favórito en:
  [GitHub] (https://github.com/jgpelaez/docker-wso2-esb/stargazers)
  [Docker Registry] (https://registry.hub.docker.com/u/jgpelaez/wso2-esb/)
