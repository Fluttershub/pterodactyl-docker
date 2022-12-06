FROM alpine:3.15

LABEL maintainer="MrFlutters"

ARG VERSION

ENV STARTUP_TIMEOUT=5 \
    PANEL_VERSION=${VERSION}

WORKDIR /var/www/html

ADD https://php.hernandev.com/key/php-alpine.rsa.pub /etc/apk/keys/php-alpine.rsa.pub

RUN apk --update-cache add ca-certificates && \
    echo "https://php.hernandev.com/v3.15/php-8.1" >> /etc/apk/repositories && \
    apk --update add curl gettext nginx php php php-bcmath php-common php-dom php-fileinfo \
    php-fpm php-gd php-memcached php-mbstring php-openssl php-pdo php-phar php-json php-sodium php-xmlwriter \
    php-pdo_mysql php-session php-curl php-simplexml php-xml php-tokenizer php-ctype php-zlib php-zip tini \
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
