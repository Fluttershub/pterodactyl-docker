FROM alpine:3.13

LABEL maintainer="MrFlutters"

ARG VERSION

ENV STARTUP_TIMEOUT=5 \
    PANEL_VERSION=${VERSION}

WORKDIR /var/www/html

ENV CURRENT_PHP_VERSION=8.1.0

ADD https://php.hernandev.com/key/php-alpine.rsa.pub /etc/apk/keys/php-alpine.rsa.pub

RUN apk --update-cache add ca-certificates && \
    echo "@php https://php.hernandev.com/v3.13/php-8.1" >> /etc/apk/repositories && \
    apk --update add curl gettext nginx php8@php php8-bcmath@php php8-common@php php8-dom@php php8-fileinfo@php \
    php8-fpm@php php8-gd@php php8-mbstring@php php8-openssl@php php8-pdo@php php8-phar@php php8-json@php php8-sodium@php php8-xmlwriter@php \
    php8-pdo_mysql@php php8-session@php php8-curl@php php8-simplexml@php php8-xml@php php8-tokenizer@php php8-ctype@php php8-zlib@php php8-zip@php tini \
    && mkdir -p /var/www/html /run/nginx /etc/nginx/conf.d/
 
RUN \
 curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/download/${PANEL_VERSION}/panel.tar.gz \
 && tar -xzvf panel.tar.gz \
 && rm panel.tar.gz \
 && chmod -R 755 storage/* bootstrap/cache \
 && find storage -type d > .storage.tmpl \
 && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
 && cp .env.example .env \
 && composer install --ansi --no-dev --optimize-autoloader \
 && rm .env ./storage -rf \
 && chown nginx:nginx * -R

COPY ./ /

# Remove Migration that isn't compatible
RUN rm /var/www/html/database/migrations/2020_10_10_165437_change_unique_database_name_to_account_for_server.php

RUN \
    ln -s /data/storage storage; \
    ln -s /data/pterodactyl.conf .env;

VOLUME [ "/data" ]

# Expose HTTP and HTTPS ports
EXPOSE 0 443

ENTRYPOINT [ "/sbin/tini", "--", "/entrypoint.sh" ]

CMD [ "p:start" ]
