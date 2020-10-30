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