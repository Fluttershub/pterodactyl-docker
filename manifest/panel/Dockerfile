FROM alpine:3.22

LABEL maintainer="hotarublaze <https://github.com/HotaruBlaze>"

ARG VERSION

ENV STARTUP_TIMEOUT=5 \
    PANEL_VERSION=${VERSION}

WORKDIR /app

RUN echo http://dl-cdn.alpinelinux.org/alpine/v3.22/community/ >> /etc/apk/repositories

RUN apk --update add curl gettext nginx php83 php83-bcmath php83-common php83-dom php83-fileinfo php83-posix \
    php83-fpm php83-gd php83-mbstring php83-openssl php83-pdo php83-phar php83-json php83-sodium php83-xmlwriter \
    php83-pdo_mysql php83-session php83-curl php83-simplexml php83-xml php83-tokenizer php83-ctype php83-zlib php83-zip tini \
    && mkdir -p /app /run/nginx /etc/nginx/conf.d/

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
