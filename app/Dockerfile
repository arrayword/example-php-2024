FROM php:8.2.21-fpm as prod

# 本番環境php.ini
COPY ./app/php.ini-production /usr/local/etc/php/php.ini

RUN apt-get update \
&& apt-get install -y libzip-dev \
&& apt-get install -y unzip \
&& docker-php-ext-install pdo pdo_mysql zip

# composer
COPY --from=composer /usr/bin/composer /usr/bin/composer


# 依存パッケージを事前インストールしてキャッシュ
COPY ./src/composer.* .

ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_RUNTIME_ENV=virtualbox
RUN composer install --prefer-dist --no-dev --no-scripts --no-progress --no-interaction

# アプリケーションをイメージ内にコピー
COPY ./src .

# オートローダー最適化
RUN composer dump-autoload --optimize

# Laravel最適化
RUN php artisan optimize

# 開発環境用追加ビルド
FROM prod as dev

# 開発環境php.ini
COPY ./app/php.ini-development /usr/local/etc/php/php.ini

# オートローダー非最適化
RUN composer dump-autoload

#Laravel非最適化
RUN php artisan optimize:clear

# XDebug
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/
RUN install-php-extensions xdebug
COPY ./app/conf.d/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

