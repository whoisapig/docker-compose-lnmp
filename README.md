
### 构建通过容器构建本地开发环境

**目标：** 通过docker多容器构建本地开发环境；基于alpine作为基础基础镜像，编写dockerfile文件，用于成镜像；通过docker-compose管理生成并管理容器；

**包括容器：** 这次生成构建的环境中主要包括了 **php56 + php72 + mysql7 + nginx + redis** 服务容器；每个服务单一容器，尽量实现单容器即单进程

**容器间的调用：**  如图

![image](EA6895269305439CA3280F0AFBCF9AE9)


---

### 1，基于alpine 基础镜像制作lnmp镜像

#### 制作alpine的自定义系统镜像

拉取 alpine 官方镜像
```
$ docker pull alpine
$ docker tag  alpine alpine:3.11
$ docker images "*alpine"       
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              3.11                a24bb4013296        4 months ago        5.57MB
alpine              latest              a24bb4013296        4 months ago        5.57MB
```

替换 alpine 源
```
$ cat repositories 
http://mirrors.aliyun.com/alpine/v3.11/main
http://mirrors.aliyun.com/alpine/v3.11/community
```

编写 alpine 基础镜像 Dockerfile (RUN 可以按照实际情况去掉部分)
```
FROM alpine:3.11 
LABEL maintainer="yang"
COPY etc/repositories /etc/apk/repositories 
RUN  apk update; apk add  iotop  gcc libgcc libc-dev libcurl libc-utils pcre-dev zlib-dev  libnfs make  pcre pcre2 zip unzip net-tools pstree wget libevent libevent-dev iproute2

```

编写 sh 脚本
```
#!/bin/bash
docker build -t alpine-base:3.11 .
```

执行 sh 脚本生成镜像
```
$ ./buld.sh 
$ docker images "*alpine*"
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
alpine-base         3.11                2dc2146f8933        About a minute ago   181MB
alpine              3.11                a24bb4013296        4 months ago         5.57MB
alpine              latest              a24bb4013296        4 months ago         5.57MB
```

目录展示
```
$ tree ./     
./
├── buld.sh
├── Dockerfile
└── etc
    └── repositories

1 directory, 3 files

```

#### 2，制作nginx的自定义系统镜像--by alpine

下载 nginx 源码包
```
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

准备 nginx 文件（可以用自己的，这里使用的是nginx包里面的）
```
$ tar -zxvf nginx-1.16.1.tar.gz
$ copy nginx-1.16.1/conf/nginx.conf ../etc/nginx.conf
$ copy nginx-1.16.1/html/index.html ../page/index.html
```

编写 Dockerfile 文件
```
FROM alpine-base:3.11
LABEL maintainer="yang"
ADD src/nginx-1.16.1.tar.gz /usr/local/src

RUN cd /usr/local/src/nginx-1.16.1 && ./configure  --prefix=/apps/nginx  && make  && make install
RUN ln -s /apps/nginx/sbin/nginx  /usr/bin/
RUN addgroup -g 2020 -S  nginx
RUN adduser nginx -s /sbin/nologin -S -D  -u 2020 -G nginx

COPY etc/nginx.conf /apps/nginx/conf/nginx.conf
ADD etc/vhost /apps/nginx/conf/vhost
ADD page/index.html /data/nginx/html/index.html
RUN chown  -R  nginx.nginx  /data/nginx/ /apps/nginx/

EXPOSE 80 443

CMD ["nginx","-g","daemon off;"]
```

编写 sh 脚本
```
#!/bin/bash
docker build -t nginx-alpine:1.16.1 .
```

执行 sh 脚本，生成镜像
```
$ ./build.sh
$ docker images "*alpine*"
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx-alpine        1.16.1              dd83133ca6e1        12 seconds ago      187MB
alpine-base         3.11                2dc2146f8933        About an hour ago   181MB
alpine              3.11                a24bb4013296        4 months ago        5.57MB
alpine              latest              a24bb4013296        4 months ago        5.57MB

```


目录展示：
```
$ tree ./
./
├── build.sh
├── Dockerfile
├── etc
│   ├── nginx.conf
│   └── vhost
├── page
│   └── index.html
└── src
    └── nginx-1.16.1.tar.gz

4 directories, 5 files

```

#### 3，制作 php 的自定义系统镜像--by alpine

> 基于 alpine-php 镜像，添加所需扩展

##### 制作 php7.2 镜像

> 注意：配置文件后面会映射出来

编写 dockerfile 文件
```
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

```

编写 sh 脚本
```
#!/bin/bash
docker build -t php:7.2.7-fpm-alpine3.7 .
```

执行 sh 脚本，生成镜像
```
./build.sh
```

##### 制作 php5.6 镜像


替换 alpine 源(如果下載不慢可以不更換这个源)
```
$ cat repositories 
http://mirrors.aliyun.com/alpine/v3.4/main
http://mirrors.aliyun.com/alpine/v3.4/community
```

编写 dockerfile 文件
```
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
```

编写 sh 脚本
```
#!/bin/bash
docker build -t php:5.6.30-fpm-alpine .
```

执行 sh 脚本，生成镜像
```
./build.sh
```


#### 4，Docker-Compose 文件编写

##### 编写 docker-compose.yml 文件
```
version: "3"
services:
  nginx:
    image: nginx:alpine
    container_name: dnmp-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./www/:/var/www/html/:rw
      - ./conf/nginx/vhost/:/etc/nginx/conf.d/:rw
      - ./conf/nginx/nginx.conf:/etc/nginx/nginx.conf:rw
      - ./log/nginx/:/var/log/dnmp/:rw
    networks:
      - net-php

  php:
    build: ./php/php72/
    container_name: dnmp-php
    expose:
      - "9000"
    volumes:
      - ./www/:/var/www/html/:rw
      - ./conf/php/php7/php.ini:/usr/local/etc/php/php.ini:rw
      - ./conf/php/php7/php-fpm.d/www.conf:/usr/local/etc/php-fpm.d/www.conf:rw
      - ./log/php/php7/:/var/log/dnmp/:rw
    networks:
      - net-php
      - net-mysql
      - net-redis

  mysql:
    image: mysql:5.6
    container_name: dnmp-mysql
    ports:
      - "3306:3306"
    volumes:
      - ./conf/mysql/my.cnf:/etc/mysql/my.cnf:rw
      - ./mysql/:/var/lib/mysql/:rw
      - ./log/mysql/:/var/log/dnmp/:rw
    networks:
      - net-mysql
    environment:
      MYSQL_ROOT_PASSWORD: "123456"

  redis:
    image: redis:latest
    container_name: dnmp-redis
    networks:
      - net-redis
    ports:
      - "6379:6379"

networks:
  net-php:
  net-mysql:
  net-redis:
```

##### 執行 docker-compose.yml 文件

```
$ docker-compose up 
$ docker-compose ps
        Name                      Command               State                     Ports                  
---------------------------------------------------------------------------------------------------------
base-alpine            /bin/sh                          Exit 0                                           
dnmp-mysql5.7-alpine   docker-entrypoint.sh mysqld      Up       0.0.0.0:3306->3306/tcp, 33060/tcp       
dnmp-nginx-alpine      nginx -g daemon off;             Up       0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp
dnmp-php56-alpine      docker-php-entrypoint php-fpm    Up       9000/tcp                                
dnmp-php72-alpine      docker-php-entrypoint php-fpm    Up       9000/tcp                                
dnmp-redis-alpine      docker-entrypoint.sh redis ...   Up       0.0.0.0:6379->6379/tcp   
```
可以查看到 容器正常启动；

注意：如果发现有容器启动失败 可以通过 ** logs **查看对应的** service **的日志

#### 5，容器基本构建完毕；调整对应的配置文件

##### nginx日志调整

> 调整 nginx 和 php-fpm 的ip socket （使用php56容器）
```
location ~ \.php$ {
        fastcgi_pass   php56:9000;
        fastcgi_index  index.php;
        include        fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    }

```

> 调整 nginx 和 php-fpm 的ip socket （使用php7容器）
```
location ~ \.php$ {
        fastcgi_pass   php72:9000;
        fastcgi_index  index.php;
        include        fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    }
```
##### 调整完成后重启nginx服务才会生效

