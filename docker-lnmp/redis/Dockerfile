FROM alpine-base:3.11
LABEL maintainer="yang"
ADD src/redis-6.0.8.tar.gz /usr/local/src

RUN cd /usr/local/src/redis-6.0.8 && ./configure  --prefix=/apps/redis  && make  && make install
RUN ln -s /apps/nginx/sbin/nginx  /usr/bin/
RUN addgroup -g 2020 -S  nginx
RUN adduser nginx -s /sbin/nologin -S -D  -u 2020 -G nginx

COPY etc/nginx.conf /apps/nginx/conf/nginx.conf
ADD etc/vhost /apps/nginx/conf/vhost
ADD page/index.html /data/nginx/html/index.html
RUN chown  -R  nginx.nginx  /data/nginx/ /apps/nginx/

EXPOSE 80 443

CMD ["nginx","-g","daemon off;"]