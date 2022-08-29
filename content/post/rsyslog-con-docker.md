---
title: "Rsyslog Con Docker"
date: 2022-08-28T19:21:16-05:00
draft: false
---

Rsyslog server en un contenedor docker.
---

En este post vamos a correr un contenedor con rsyslog como servidor en docker. **Rsyslog** es un servicio que se encarga de la recolección de los logs en los sistemas operativos linux.

Primero generar el archivo **Dockerfile**, como se se muestra a continuación:

```
FROM ubuntu:20.04

RUN apt update && apt install rsyslog -y

RUN echo '$ModLoad imudp \n\
$UDPServerRun 514 \n\
$ModLoad imtcp \n\
$InputTCPServerRun 514 \n\
$template RemoteStore, "/var/log/remote/%HOSTNAME%/%$year%%$Month%%$Day%.log" \n\
:source, !isequal, "localhost" -?RemoteStore \n\
:source, isequal, "last" ~ ' > /etc/rsyslog.conf

ENTRYPOINT ["rsyslogd", "-n"]
```

Después construimos la imagen.

```
$ sudo docker build -t docker-syslog .
```

Los logs de los diferentes sistemas operativos van a guardarse dentro del contenedor en la ruta */var/log/remote/{host-o-ip}/{añomesdia}.log*

Para configurar los clientes se edita archivo, **/etc/rsyslog.conf** (Esto por lo general es un estándar pero en algunos casos la ruta será diferente), en la última línea se coloca la ip del host donde corre el contenedor, como se muestra a continuación.

```
*.* @<ip-del-host-del-contenedor>:514
```

Esto enviará todos los logs del cliente al contenedor, en la ruta antes mencionada.

Con los clientes configurados lanzamos el contenedor.

```
# sudo docker run --cap-add SYSLOG -p 514:514 -p 514:514/udp --name rsyslog docker-syslog
```

Aquí puede hacer las modificaciones necesarias según su caso. 

Con esto se finaliza el proceso ;)

Visita el repositorio de este proyecto [docker-syslog-server](https://github.com/nanyhel/docker-rsyslog-server)