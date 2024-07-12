FROM php:8.1-apache

RUN apt-get update -qq \
  && apt-get install wget mariadb-client -yq

RUN a2enmod rewrite

ADD vhost.conf /etc/apache2/sites-available/000-default.conf

RUN a2ensite 000-default

# install the PHP extensions we need
RUN apt-get update && apt-get install -y libpng-dev libjpeg-dev  libwebp-dev libpq-dev libzip-dev zip libonig-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure zip \
    && docker-php-ext-configure gd --with-jpeg --with-webp \
    && docker-php-ext-install gd opcache pdo pdo_mysql pdo_pgsql zip mysqli \
    && docker-php-ext-install  mbstring
# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=0'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Install MailHog Sendmail support.
RUN apt-get update -qq && apt-get install -yq git golang-go &&\
    mkdir -p /opt/go &&\
    export GOPATH=/opt/go &&\
    go env -w GO111MODULE=off &&\
    go get github.com/mailhog/mhsendmail

# Copy PHP configs.
COPY conf.d/* /usr/local/etc/php/conf.d/
RUN chmod 644 /usr/local/etc/php/conf.d/*

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

# Install drush.
RUN composer global require drush/drush:12

# Add Composer bin to PATH
ENV PATH="/root/.composer/vendor/bin:${PATH}"

WORKDIR /var/www/web
