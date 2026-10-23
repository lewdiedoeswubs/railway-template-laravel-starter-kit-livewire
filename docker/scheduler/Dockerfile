FROM serversideup/php:8.5-cli-v4.2.1 AS base

USER root

# Install additional PHP extensions
RUN install-php-extensions intl

# Composer dependencies stage
FROM base AS composer-dependencies

COPY composer.json composer.lock ./

RUN composer install --no-autoloader --no-dev --no-interaction --no-scripts --prefer-dist

# Release stage
FROM base AS final

# ServerSideUp environment settings - https://serversideup.net/open-source/docker-php/docs/reference/environment-variable-specification
ENV AUTORUN_ENABLED=true \
    AUTORUN_LARAVEL_MIGRATION=false \
    AUTORUN_LARAVEL_STORAGE_LINK=false \
    PHP_OPCACHE_ENABLE=1

# APP files + pre-built vendor folder from cache layer
COPY --chown=www-data:www-data . .
COPY --chown=www-data:www-data --from=composer-dependencies $APP_BASE_DIR/vendor ./vendor

# Regenerate optimized autoloader
RUN composer install --no-dev --no-interaction --optimize-autoloader --prefer-dist

CMD ["php", "/var/www/html/artisan", "schedule:work"]

# Switch back to www-data
USER www-data
