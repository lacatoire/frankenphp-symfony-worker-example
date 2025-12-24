ARG PHP_VERSION=8.4

FROM dunglas/frankenphp:php${PHP_VERSION}-alpine AS php-builder

RUN apk add --no-cache \
  autoconf \
  make \
  g++ \
  libzip-dev \
  icu-dev \
  zlib-dev \
  openssl-dev \
  libintl \
  $PHPIZE_DEPS \
 && docker-php-ext-configure intl \
 && docker-php-ext-install \
    intl \
    zip \
    pdo_mysql \
    calendar \
    bcmath \
    opcache \
    pcntl \
 && pecl install apcu pcov \
 && docker-php-ext-enable apcu pcov

FROM dunglas/frankenphp:php${PHP_VERSION}-alpine AS php-runtime

RUN apk add --no-cache \
  icu-libs \
  libzip \
  zlib \
  openssl \
  libintl \
  bash

ARG USER_UID=1000
WORKDIR /srv/app

# Récupère extensions compilées
COPY --from=php-builder /usr/local/lib/php/extensions /usr/local/lib/php/extensions
COPY --from=php-builder /usr/local/etc/php/conf.d /usr/local/etc/php/conf.d

# Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# User
RUN deluser --remove-home www-data || true \
    && addgroup -S www-data \
    && adduser -S -G www-data -u $USER_UID www-data

RUN mkdir -p /etc/caddy /data/caddy/locks /config/caddy \
    && touch /data/caddy/instance.uuid \
    && chown -R www-data:www-data /data /config /etc/caddy /srv/app

# Your Caddyfile (mounted later, but container must have path)
COPY .docker/frankenphp/php-ini-overrides.ini $PHP_INI_DIR/conf.d/

EXPOSE 8080

FROM php-runtime AS dev

RUN apk add --no-cache bash curl git nano

USER www-data

ENV APP_ENV=dev

COPY .docker/frankenphp/Caddyfile /etc/caddy/Caddyfile

CMD ["frankenphp", "run", "--config=/etc/caddy/Caddyfile", "--adapter=caddyfile"]

FROM php-runtime AS prod
RUN apk add --no-cache curl git

ENV APP_ENV=prod
# Your Caddyfile (mounted later, but container must have path)
COPY .docker/frankenphp/Caddyfile /etc/caddy/Caddyfile

# Code applicatif
COPY bin /srv/app/bin/
COPY config /srv/app/config/
COPY migrations /srv/app/migrations/
COPY public /srv/app/public/
COPY src /srv/app/src/
COPY templates /srv/app/templates/
COPY .env.dist /srv/app/.env
COPY composer.json composer.lock symfony.lock /srv/app/

RUN chown -R www-data:www-data /srv/app

USER www-data

# Install prod deps + warmup
RUN COMPOSER_MEMORY_LIMIT=-1 composer install \
        --no-dev \
        --optimize-autoloader \
        --no-interaction \
        --prefer-dist \
    && php bin/console cache:warmup -e prod \
    && php bin/console assets:install

CMD ["frankenphp", "run", "--config=/etc/caddy/Caddyfile", "--adapter=caddyfile"]