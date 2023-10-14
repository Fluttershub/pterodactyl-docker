FROM alpine:3.18

LABEL maintainer="hotarublaze <https://github.com/HotaruBlaze>"

ARG VERSION

ENV STARTUP_TIMEOUT=5 \
    PANEL_VERSION=${VERSION}

WORKDIR /var/www/html

RUN echo http://dl-cdn.alpinelinux.org/alpine/v3.18/community/ >> /etc/apk/repositories

RUN apk --update add curl gettext nginx php81 php81-bcmath php81-common php81-dom php81-fileinfo php81-posix \
    php81-fpm php81-gd php81-mbstring php81-openssl php81-pdo php81-phar php81-json php81-sodium php81-xmlwriter \
    php81-pdo_mysql php81-session php81-curl php81-simplexml php81-xml php81-tokenizer php81-ctype php81-zlib php81-zip tini \
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

RUN \
    dos2unix /entrypoint.sh; \
    dos2unix /usr/local/bin/wait-for;

VOLUME [ "/data" ]

# Expose HTTP and HTTPS ports
EXPOSE 0 443

ENTRYPOINT [ "/sbin/tini", "--", "/entrypoint.sh" ]

CMD [ "p:start" ]
