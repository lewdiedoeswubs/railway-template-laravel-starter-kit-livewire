FROM serversideup/php:8.5-frankenphp-v4.2.1 AS base

USER root

# Install additional PHP extensions
RUN install-php-extensions intl

# Composer dependencies stage
FROM base AS composer-dependencies

COPY composer.json composer.lock ./

RUN composer install --no-autoloader --no-dev --no-interaction --no-scripts --prefer-dist

# Node tooling stage
FROM base AS node-runtime

# Install Node.js LTS
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && docker-php-serversideup-dep-install-debian nodejs

# JS dependencies stage
FROM node-runtime AS npm-dependencies

ARG FONTAWESOME_NPM_AUTH_TOKEN

# Copy only manifest files before installing dependencies
COPY package.json package-lock.json ./

# Install npm dependencies
RUN npm ci

# Asset build stage
FROM npm-dependencies AS frontend-build

ARG FONTAWESOME_NPM_AUTH_TOKEN
ARG RAILWAY_GIT_COMMIT_SHA
ARG SENTRY_AUTH_TOKEN

# Copy files and dependencies for Wayfinder route generation
COPY . .
COPY --from=composer-dependencies $APP_BASE_DIR/vendor ./vendor

RUN composer install --no-dev --no-interaction --optimize-autoloader --prefer-dist

# Build frontend assets
RUN npm run build

# Release stage
FROM base AS final

# ServerSideUp environment settings - https://serversideup.net/open-source/docker-php/docs/reference/environment-variable-specification
ENV AUTORUN_ENABLED=true \
    AUTORUN_LARAVEL_STORAGE_LINK=false \
    HEALTHCHECK_PATH="/up" \
    PHP_OPCACHE_ENABLE=1

# APP files + pre-built vendor folder from cache layer
COPY --chown=www-data:www-data . .
COPY --chown=www-data:www-data --from=composer-dependencies $APP_BASE_DIR/vendor ./vendor
COPY --chown=www-data:www-data --from=frontend-build $APP_BASE_DIR/public/build ./public/build

# Regenerate optimized autoloader
RUN composer install --no-dev --no-interaction --optimize-autoloader --prefer-dist

# Switch back to www-data
USER www-data
