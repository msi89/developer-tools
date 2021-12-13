FROM php:8.1-fpm


# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # install git iproute2, procps, lsb-release (useful for CLI installs)
    && apt-get -y install git openssh-client less iproute2 procps iproute2 lsb-release zip unzip \
    #
    # Install xdebug
    && yes | pecl install xdebug \
    && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # For mbstring
    && apt-get -y install libonig-dev \
    # for gd
    && apt-get install -y zlib1g-dev libpng-dev libwebp-dev libjpeg62-turbo-dev libfreetype6-dev \
    # For PDO,zip,xml
    &&apt-get install -y  libpq-dev  libzip-dev libxml2-dev \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Install opcache
RUN docker-php-ext-install opcache

# Install pdo
RUN docker-php-ext-install pdo pgsql pdo_pgsql mysqli pdo_mysql

# Install mbstring
RUN docker-php-ext-install mbstring

# Install bcmath
RUN docker-php-ext-install bcmath

# Install tokenizer
RUN docker-php-ext-install tokenizer

# Install xml
RUN docker-php-ext-install xml

# Install zip
RUN docker-php-ext-install zip

# Install sockets
RUN docker-php-ext-install sockets

# Install exif
RUN docker-php-ext-install exif

# Install pcntl
RUN docker-php-ext-install pcntl

# Install composer
RUN curl -s http://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer

# Install redis
RUN pecl install -o -f redis
RUN rm -rf /tmp/pear
RUN docker-php-ext-enable redis

# Install gd
RUN docker-php-ext-configure gd \
    --with-freetype=/usr/include/ \
    --with-webp=/usr/include/ \
    --with-jpeg=/usr/include/;
RUN docker-php-ext-install gd

# Install nodejs
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get install -y nodejs

# Install yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt update
RUN apt install -y yarn

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# php-fpm.sock
RUN mkdir -p /var/run/php-fpm
COPY ./conf/www.conf /usr/local/etc/php-fpm.d/zzz-www.conf

# composer command
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# node command
COPY --from=node /usr/local/bin /usr/local/bin
# npm command
COPY --from=node /usr/local/lib /usr/local/lib
# yarn command
COPY --from=node /opt /opt
