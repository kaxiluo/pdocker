FROM redis:latest

#RUN set -xe \
#    && sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
#    && sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
#    && sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list

#RUN set -xe \
#    && apt-get update -yqq \
#    && apt-get install -yqq apt-utils procps vim
    
## For security settings uncomment, make the dir, copy conf, and also start with the conf, to use it
RUN mkdir -p /usr/local/etc/redis
COPY redis.conf /usr/local/etc/redis/redis.conf

VOLUME /data

EXPOSE 6379

CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
#CMD ["redis-server"]