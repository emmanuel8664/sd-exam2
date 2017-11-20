# Examen 2 #

Universidad ICESI
Curso: Sistemas Distribuidos
Docente: Daniel Barragán C.
Tema: Automatización de infraestructura (Docker)
Correo: daniel.barragan at correo.icesi.edu.co

Objetivos

•	Realizar de forma autónoma el aprovisionamiento automático de infraestructura

•	Diagnosticar y ejecutar de forma autónoma las acciones necesarias para lograr infraestructuras estables

•	Integrar servicios ejecutándose en nodos distintos

Prerrequisitos

•	Docker

•	docker-compose

El stack ELK es un paquete de tres herramientas open source de la empresa Elastic. Las herramientas son Elasticsearch, Logstash y Kibana. Estas tres herramientas son proyectos independientes pero pueden ser usadas en conjunto para desplegar un ambiente de monitoreo de infraestructura.
Deberá realizar el aprovisionamiento de un ambiente compuesto por los siguientes elementos:

•	En un servidor ejecutar: Un contenedor encargado de almacenar logs por medio de la aplicación Elasticsearch, un contenedor con la herramienta encargada de visualizar la información de los logs por medio de la aplicación Kibana

•	En uno o varios servidores ejecutar: un contenedor web y un contenedor encargado de hacer la conversión de logs por medio de la aplicación Fluentd
 
![image](https://user-images.githubusercontent.com/17281732/32998030-77257c20-cd65-11e7-88b8-e3678a3225f4.png)

En el repositorio de github del curso se encuentran ejemplos de docker y docker-compose los cuales pueden ser consultados para construir su solución.


***Consigne los comandos de Linux necesarios para el aprovisionamiento de los servicios solicitados. 
En este punto no debe incluir archivos tipo Dockerfile solo se requiere que usted identifique los comandos o acciones que debe automatizar***

```
docker run -d --name=elastic elasticsearch
```
```
docker run -d --name=kibana --link elastic:elasticsearch -p 5601:5601 kibana
```


```
FROM fluent/fluentd:v0.12
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri",
"--version", "1.9.5"]
```
```
docker build -t fluentd-elastic .
```

```
/fluentd/conf:/fluentd/etc -e FLUENTD_CONF=fluent.conf fluentd-elastic
```
```
docker run -d -p 80:80 --name=apache --log-driver=fluentd --log-opt fluentdaddress=172.17.0.4:24224 --log-opt tag=httpd.access httpd
```

***Escriba los archivos Dockerfile para cada uno de los servicios solicitados junto con los archivos fuente necesarios.
Tenga en cuenta consultar buenas prácticas para la elaboración de archivos Dockerfile.***

```
FROM fluent/fluentd:v0.12
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri",
"--version", "1.9.5"]
```

***Escriba el archivo docker-compose.yml necesario para el despliegue de la infraestructura.
No emplee configuraciones deprecated.***

```
version: '2'
services:
web:
image: httpd
ports:
- "80:80"
links:
- fluentd
logging:
driver: "fluentd"
options:
fluentd-address: localhost:24224
tag: httpd.access
fluentd:
build: ./fluentd
volumes:
- ./fluentd/conf:/fluentd/etc
links:
- "elasticsearch"
ports:
- "24224:24224"
- "24224:24224/udp"
elasticsearch:
image: elasticsearch
kibana:
image: kibana
links:
- "elasticsearch"
ports:
- "5601:5601"
```

***Incluya evidencias que muestran el funcionamiento de lo solicitado.***

![funcionamiento](https://user-images.githubusercontent.com/17281732/32998339-48cab1d0-cd68-11e7-98f2-c066d1208b3f.jpeg)


***Documente algunos de los problemas encontrados y las acciones efectuadas para su solución al aprovisionar
la infraestructura y aplicaciones.***


Cuando se quieren enviar Logs a fluentd es necesario conocer la ip del contenedor de fluentd.  
En principio, se intentó usar la opción de “link” de docker pero no dio resultado. 
Como solución se decidió exponer el servicio de fluentd a localhost


![image](https://user-images.githubusercontent.com/17281732/32998399-b8cb07d2-cd68-11e7-8c4f-bb77eb664ad2.png)

 y hacer referencia a este en vez de al contenedor. 
 
 ![image](https://user-images.githubusercontent.com/17281732/32998431-f8247bd4-cd68-11e7-8100-fe81f1c1efa2.png)

