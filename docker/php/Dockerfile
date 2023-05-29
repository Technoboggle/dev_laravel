# docker/php/Dockerfile
FROM php:8.1-fpm
#FROM 8.1-fpm-alpine3.18

RUN apt-get update
RUN apt-get install -y openssl zip unzip git curl libzip-dev libonig-dev libicu-dev autoconf pkg-config libssl-dev libsodium-dev bzip2 libbz2-dev

RUN docker-php-ext-install bcmath mbstring intl bz2 sodium zip opcache pdo pdo_mysql mysqli

RUN pecl install xdebug
RUN pecl install mongodb

COPY ./mongodb.ini "/usr/local/etc/php/conf.d/docker-php-ext-mongodb.ini"
COPY ./xdebug.ini "/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini"

# RUN addgroup -g 1000 laravel && adduser -G laravel -g laravel -s /bin/sh -D laravel

# USER laravel
