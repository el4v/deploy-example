# syntax=docker/dockerfile:1.0.0-experimental

ARG BASE_APP_IMAGE=base
ARG COMPOSER_NO_DEV

FROM registry.example.com/library/php-fpm-nginx:7.3-bionic as base

LABEL maintainer="Egor Lyutov <lyutoved@gmail.com>"

RUN apt-get update \
    && apt-get -y --no-install-recommends install \
    php7.3-mysql \
    php7.3-intl \
    php7.3-dev \
    php7.3-gd \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

FROM ${BASE_APP_IMAGE} as app

WORKDIR /web/src
RUN mkdir -p /web/src/var/cache/prod /web/src/var/dumps /web/src/var/log
COPY build/nginx/nginx.conf /etc/nginx/nginx.conf
COPY build/php/conf.d/user-config.ini /etc/php/7.3/fpm/conf.d/user-config.ini
COPY --chown=www-data src/composer.lock src/composer.json /web/src/
RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
RUN --mount=type=cache,target=/root/.composer --mount=type=ssh,required composer install --no-scripts --no-autoloader ${COMPOSER_NO_DEV}
COPY --chown=www-data . /web
RUN chown www-data:www-data -R /web/src/var
RUN composer dump-autoload --no-scripts --optimize ${COMPOSER_NO_DEV}
WORKDIR /web
