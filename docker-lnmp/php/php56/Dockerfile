FROM php:5.6.30-fpm-alpine
#COPY source/repositories /etc/apk/repositories
RUN apk add --no-cache --virtual .build-deps \
        autoconf \
        file \
        gcc \
        g++ \
        libc-dev \
        make \
        pkgconf \
        re2c \
        tzdata \
    && apk add --no-cache --virtual .run-deps \
        coreutils \
        libltdl \
        freetype-dev \
        gettext-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        curl-dev \
        libmcrypt-dev \
        libxml2-dev \
        cyrus-sasl-dev \
        libmemcached-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install -j$(nproc) \
        iconv mcrypt gettext curl mysqli pdo pdo_mysql zip \
        mbstring bcmath opcache xml simplexml sockets hash soap \
    && docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && pecl install redis-3.1.0 \
    && pecl install memcached-2.2.0 \
    && pecl install xdebug-2.5.0 \
    && docker-php-ext-enable redis memcached xdebug \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" >  /etc/timezone \
    && apk del .build-deps