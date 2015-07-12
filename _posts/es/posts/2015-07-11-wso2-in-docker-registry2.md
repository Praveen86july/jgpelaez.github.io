---
layout:     post
title: Utilizando el registro de Docker para las imagenes de WSO2
subtitle:   ""
date:       2015-07-11 12:00:00
author:     "Juan Carlos G. Peláez"
comments: true
---

Una vez que conocemos como generar una imagen de Docker, si quieres compartir esta imagen con tu equipo, existe la posibilidad de exportarla a un archivo tar, o se puede usar el archivo Dockerfile para construir de nuevo la imagen en cada uno de los docker host donde va a ser utilizada. Tener que reconstruir la imagen en el pc de cada uno de los componentes de tu equipo no es algo práctico. ¿No sería mejor si pudiéramos descargar imágenes ya listas para usar? Para resolver este problema disponemos de un elemento que docker denomina registro, donde podemos subir y bajar nuestras imágenes.

Una de las grandes ventajas que nos permitirá el registro es el de poder testear las diferentes versiones de un producto con la sola modificación de la imagen a descargar si estas están en el registro.

Podemos instalar un registro privado o bien utilizar el [registro público disponible en Docker](https://registry.hub.docker.com/). Este registro puede ser utilizado por cualquier usuario, en este post vamos a explicar como subir nuestrar imágenes de WSO2 al registro.

### Prerequisitos
...
.

Primero crearemos un repositorio público en github o bitbucket, o bien hacemos un [fork de uno existente] (https://github.com/jgpelaez/docker-wso2-esb.git).

A este repositorio añadiremos el fichero/s Dockerfile:

```docker
FROM java:openjdk-8-jre

# Maintainer
MAINTAINER Juan Carlos García Peláez

ENV WSO2_URL=http://dist.wso2.org/maven2/org/wso2/esb/wso2esb
ENV WSO2_SOFT_VER=4.8.0
RUN  \
	mkdir -p /opt/wso2 && \
	wget -P /opt/wso2 $WSO2_URL/$WSO2_SOFT_VER/wso2esb-$WSO2_SOFT_VER.zip && \
    unzip /opt/wso2/wso2esb-$WSO2_SOFT_VER.zip -d /opt/wso2 && \
    rm /opt/wso2/wso2esb-$WSO2_SOFT_VER.zip

EXPOSE 9443
CMD ["/opt/wso2esb-4.8.0/bin/wso2server.sh"]
```

Crearemos un usuario en el registro de docker https://registry.hub.docker.com/.

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/Screenshot from 2015-07-11 20:33:25.png)


Podemos utilizar nuestro usuario de GitHub para realizar el registro.

Una vez creada la cuenta podremos crear un nuevo repositorio, con construcción automática:

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/Screenshot from 2015-07-11 20:36:43.png)

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/Screenshot from 2015-07-11 20:37:31.png)

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/Screenshot from 2015-07-11 20:38:00.png)

Podremos selecionar el directorio en el que se encuentra el Dockerfile. En nuestro caso añadiremos varios Dockerfile para cada versión del ESB de WSO2.

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/Screenshot from 2015-07-11 20:38:49.png)

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/Screenshot from 2015-07-11 23:50:01.png)





## TODO

### TODO

TODO

