FROM php:8.3.37-fpm-alpine3.24

LABEL maintainer="hotarublaze <https://github.com/HotaruBlaze>"

ARG VERSION

ENV STARTUP_TIMEOUT=5 \
    PANEL_VERSION=${VERSION}

WORKDIR /app

RUN apk --update add --no-cache \
    curl gettext dos2unix 'nginx>=1.30.7-r0' tini \
    libjpeg-turbo libpng freetype libzip \
    && apk add --no-cache --virtual .build-deps \
        libjpeg-turbo-dev libpng-dev freetype-dev libzip-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j"$(nproc)" bcmath gd pdo_mysql zip \
    && apk del .build-deps \
    && mkdir -p /app /run/nginx /etc/nginx/conf.d/

RUN nginx -v && php -v

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
RUN rm /app/database/migrations/2020_10_10_165437_change_unique_database_name_to_account_for_server.php

RUN \
    ln -s /data/storage storage; \
    ln -s /data/pterodactyl.conf .env;

RUN \
    dos2unix /entrypoint.sh; \
    dos2unix /usr/local/bin/wait-for;

VOLUME [ "/data" ]

# Expose HTTP
EXPOSE 80

ENTRYPOINT [ "/sbin/tini", "--", "/entrypoint.sh" ]

CMD [ "p:start" ]
