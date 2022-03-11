# **Práctica 16**: «Dockerizar» una aplicación LAMP.

En esta práctica tendremos que crear un archivo Dockerfile para crear una imagen Docker que contenga una aplicación web LAMP. Posteriormente deberá realizar la implantación del sitio web en Amazon Web Services (AWS) haciendo uso de contenedores Docker y de la herramienta Docker Compose.

## Requisitos de la instancia.

Debemos disponer de una instancia EC2 con los siguientes puertos abiertos

    80
    8080
    443
    3306

## Instalación de Docker y docker-compose.

Nuestr máquina debe disponer del servicio Docker y Docker-Compose, para instalarlos lanzamos las siguientes líneas de comandos:

    curl -fsSL https://get.docker.com -o get-docker.sh

    sudo sh get-docker.sh

    usermod -aG docker $USERNAME

    systemctl start docker

    systemctl enable docker

    newgrp docker

    apt install docker-compose

## Archivo dockerfile

Creamos un archivo Dockerfile para crear una imagen que contenga el servicio de Apache:

    FROM ubuntu:21.04

    LABEL AUTHOR="Manuel Crisol Belver"

    ENV DEBIAN_FRONTEND nointeractive

    RUN apt update && \
        apt install -y apache2 && \
        apt install -y php && \
        apt install -y libapache2-mod-php && \
        apt install -y php-mysql && \
        rm -rf /var/lib/apt/lists/*

    ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

## Archivo docker-compose.yml

Creamos el siguiente archivo docker-compose.yml para poder desplegar los servicios de Apache, MySQL y PhpMyAdmin:

    version: '3.3'

    services:
    apache:
        build: ./apache
        ports:
        - 80:80 
        restart: always
        depends_on:
        - mysql
        volumes:
        - ./iaw-practica-lamp/src:/var/www/html
        networks:
        - frontend_network
        - backend_network

    mysql:
        image: mysql:8.0
        environment:
        - MYSQL_ROOT_PASSWORD=root
        - MYSQL_DATABASE=lamp_db
        - MYSQL_USER=lamp_user
        - MYSQL_PASSWORD=lamp_password
        restart: always
        volumes:
        - mysql_data:/var/lib/mysql
        - ./iaw-practica-lamp/db:/docker-entrypoint-initdb.d
        networks:
        - backend_network
        security_opt:
        - seccomp:unconfined

    phpmyadmin:
        image: phpmyadmin:5
        restart: always
        ports:
        - 8080:80
        environment:
        - PMA_HOST=mysql
        depends_on:
        - mysql
        networks:
        - frontend_network

    volumes:
    mysql_data:

    networks:
    frontend_network:
    backend_network:

## Comprobamos que funciona.

Por último lanzamos los conetenedores con docker-compose:

    docker-compose-up

Tras hacer esto ya nos debería funcionar, y al entrar a la IP de la instancia debería salirnos la base de datos y dejarnos añadir datos.

<img src="/images/aws1.png">

Comprobamos también que podemos acceder a PHPMyAdmin:

<img src="/images/aws2.png">