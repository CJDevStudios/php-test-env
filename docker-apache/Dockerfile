ARG BASE_IMAGE=php:8.2-apache


#####
# Fetch composer latest build
#####
FROM composer:latest AS composer

#####
# Build main image
#####
FROM $BASE_IMAGE

LABEL \
  org.opencontainers.image.title="Cypress Testing Env - Apache" \
  org.opencontainers.image.description="This container is used to run a PHP application using Apache for Cypress tests" \
  org.opencontainers.image.url="https://github.com/CJDevStudios/cypress-test-env" \
  org.opencontainers.image.source="git@github.com:CJDevStudios/cypress-test-env"

RUN \
  apt update \
  \
  # Install exif extension.
  && docker-php-ext-install exif \
  \
  # Install GD PHP extension.
  && apt install -y zlib1g-dev libjpeg-dev libpng-dev libwebp-dev libfreetype6-dev \
  && docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
  && docker-php-ext-install gd \
  \
  # Install intl PHP extension.
  && apt install -y libicu-dev \
  && docker-php-ext-install intl \
  \
  # Install ldap PHP extension.
  && apt install -y libldap-dev \
  && docker-php-ext-install ldap \
  \
  # Install mysqli PHP extension.
  && docker-php-ext-install mysqli \
  \
  # Install pcntl PHP extension (required for composer-require-checker).
  && docker-php-ext-install pcntl \
  \
  # Install Zip PHP extension.
  && apt install -y libzip-dev \
  && docker-php-ext-install zip \
  \
  # Install APCU PHP extension.
  && pecl install apcu \
  && docker-php-ext-enable apcu \
  && echo "apc.enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini \
  && echo "apc.enable_cli=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini \
  \
  # Update PHP configuration.
  && echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/docker-php-memory.ini \
  \
  # Disable sodium PHP extension (Test should validate that polyfill works).
  && rm /usr/local/etc/php/conf.d/docker-php-ext-sodium.ini \
  \
  # Install nodejs and npm.
  && apt install -y nodejs npm \
  \
  # Install git and zip used by composer when fetching dependencies.
  && apt install -y git unzip \
  \
  # Install gettext and perl used to validate locales extraction.
  && apt install -y gettext \
  && apt install -y perl \
  # Install sudo that may be usefull to temporarly install upcoming required system components.
  && apt install -y sudo \
  && (echo "apptest  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/apptest) \
  \
  # Install bash that is used in some scripts.
  && apt install -y bash \
  \
  # Install patch utility that may be usefull to patch dependencies.
  && apt install -y patch \
  # Install Cypress dependencies
  && apt install -y libgtk2.0-0 libgtk-3-0 libgbm-dev libnotify-dev libnss3 libxss1 libasound2 libxtst6 xauth xvfb

# Copy composer binary
COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN addgroup -gid 1000 apptest \
  && useradd -d /home/apptest -g apptest -u 1000 apptest \
  && mkdir -p /var/apptest \
  && chown apptest:apptest /var/apptest
VOLUME /home/apptest
VOLUME /var/apptest
# Remove default Apache configuration
RUN rm /etc/apache2/sites-enabled/000-default.conf
# Copy Apache configuration
COPY apache-sites/apptest-vhost.conf /etc/apache2/sites-enabled/apptest.conf
# Enable Apache modules
RUN a2enmod rewrite

USER apptest
WORKDIR /var/apptest
