FROM alpine:3.11 
LABEL maintainer="yang"
COPY etc/repositories /etc/apk/repositories 
RUN  apk update; apk add  iotop  gcc libgcc libc-dev libcurl libc-utils pcre-dev zlib-dev  libnfs make  pcre pcre2 zip unzip net-tools pstree wget libevent libevent-dev iproute2
