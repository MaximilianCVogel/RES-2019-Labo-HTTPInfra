FROM php:7.0-apache

RUN apt-get update &&\
    apt-get install -y vim

COPY apache2-foreground /usr/local/bin/
COPY conf/ /etc/apache2
COPY templates /var/apache2/templates

RUN a2enmod proxy proxy_http
RUN a2ensite 000-* 001-*

 
