---
layout:     post
title: Utilizando el registro de Docker para las imagenes de WSO2
subtitle:   ""
date:       2015-07-11 12:00:00
author:     "Juan Carlos G. Peláez"
comments: true
---

## Introducción

Una vez que conocemos como [generar una imagen de Docker] (https://itscenario.wordpress.com/2014/11/09/dockerizing-wso2-esb/), si queremos compartir esta imagen con el equipo, o bien utilizarla para diferentes entornos, existe la posibilidad de exportarla a un archivo tar, o se puede usar el archivo [Dockerfile] (https://docs.docker.com/reference/builder/) para construir de nuevo la imagen en cada uno de los docker host donde va a ser utilizada. 
Tener que reconstruir la imagen en el pc de cada uno de los componentes de tu equipo no es algo práctico. ¿No sería mejor si pudiéramos descargar imágenes ya listas para usar? Para resolver este problema disponemos de un elemento que docker denomina registro, donde podemos subir y bajar nuestras imágenes.

Una de las grandes ventajas que nos permitirá el registro es el de poder testear las diferentes versiones de un producto con la sola modificación de la imagen a descargar si estas están en el registro.

Podemos instalar un registro privado o bien utilizar el [registro público disponible en Docker](https://registry.hub.docker.com/). Este registro puede ser utilizado por cualquier usuario, y podemos realizar búsquedas para comprobar si otro usuario ha añadido la imagen que deseamos.

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/search-docker-registry.png)

Si vamos a utilizar una imagen subida al registro por otro usuario es recomendable utilizar las marcadas con "construcción automática". En estas imágenes podemos comprobar como han sido creadas con el [Dockerfile] (https://docs.docker.com/reference/builder/), si han sido subidas directamente por un usuario no debemos confiar en ella si no es un usuario de confianza, ya que es una imagen que puede contener código malicioso.

Para el uso en producción de Docker es recomendable crear nuestras propias imágenes ya que las imágenes del registro pueden ser eliminadas por el usuario que las ha subido. También se recomienda usar las imágenes marcadas como oficiales, estas han sido elaboradas bien por el vendedor, por el equipo de Docker, o por usuarios con mucha popularidad.

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/docker-official-repo.png)

En este post vamos a explicar como subir nuestrar imágenes de WSO2 al registro.

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

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/docker-registry-signup.png)


Podemos utilizar nuestro usuario de GitHub para realizar el registro.

Una vez creada la cuenta podremos crear un nuevo repositorio, con construcción automática, el registro leerá nuestro [Dockerfile] (https://docs.docker.com/reference/builder/) y automáticamente realizará la construcción:

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/docker-registry-repositories.png)

Elegimos nuestra cuenta de bitbucket o de github:

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/docker-registry-git-source.png)

Y seleccionamos el repositorio en el que se encuentra el/los [Dockerfile] (https://docs.docker.com/reference/builder/)/s:

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/docker-registry-git-source-repository.png)

Podremos selecionar el directorio en el que se encuentra el [Dockerfile] (https://docs.docker.com/reference/builder/). En nuestro caso añadiremos varios [Dockerfile] (https://docs.docker.com/reference/builder/) para diferentes versiones del ESB de WSO2. 
También podremos cambiar el nombre del repositorio, la convención que utilizamos es para el repositorio en git docker-[vendedor]-[software] y para el registro de docker [vendedor]-[software].

![Creación del repositorio](/media/2015-07-11-wso2-in-docker-registry/docker-registry-repository-creation.png)

La construcción se puede ejecutar manualmente, aunque normalmente no será necesario, ya que cuando modificamos un fichero en nuestro repositoria de git, se ejecuta automáticamente la construcción.

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/docker-registry-repository-build-details.png)

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

![registro en docker](/media/2015-07-11-wso2-in-docker-registry/docker-wso2-esb-admin.png)

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
