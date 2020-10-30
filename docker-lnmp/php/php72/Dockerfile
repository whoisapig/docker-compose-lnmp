FROM php:7.2.7-fpm-alpine3.7

# 使用的是alpine的edge作为php容器的内核
# php的版本为7.2.7
# 镜像默认自带的扩展为：Core,ctype,curl,date,dom,fileinfo,filter,ftp,hash,iconv,json,libxml,mbstring,mysqlnd,openssl,pcre,PDO,pdo_sqlite,Phar,posix,readline,Reflection,session,SimpleXML,sodium,SPL,sqlite3,standard,tokenizer,xml,xmlreader,xmlwriter,zlib

# alpine3.7没有make命令，甚至没有gcc、g++，虽然有pecl，但是由于没有gcc编译器，所以也不能运行phpize
# 但是/usr/local/bin目录下有一个docker-php-ext-install程序专门用来安装php扩展。
# 使用方式：docker-php-ext-install extension_name


# Notice:
# 1. Mcrypt was DEPRECATED in PHP 7.2.6, and REMOVED in PHP 7.2.0.
# 2. opcache requires PHP version >= 7.0.0.
# 3. soap requires libxml2-dev.
# 4. xml, xmlrpc, wddx require libxml2-dev and libxslt-dev.
# 5. Line `&& :\` is just for better reading and do nothing.

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
    && docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && pecl install redis-4.0.2 \
    && pecl install memcached-3.0.4 \
    && pecl install xdebug-2.6.0 \
    && docker-php-ext-enable redis memcached xdebug \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" >  /etc/timezone \
    && apk del .build-deps


# Composer
#RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \



