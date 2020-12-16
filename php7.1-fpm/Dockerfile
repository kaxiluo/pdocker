FROM php:7.1-fpm-alpine3.9
##todo 合并php

# apk 是 alpine 的一个包管理器
RUN set -ex \
    # 在临时目录进行这一切
    && cd /tmp \
    # 把 apk 的默认源改为aliyun镜像
    && sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    # 更新包列表
    && apk update \
    && apk add --no-cache vim git autoconf openssl-dev build-base zlib-dev re2c libpng-dev oniguruma-dev imap-dev

COPY ext/redis-5.3.2.tgz /tmp/redis-5.3.2.tgz
COPY ext/igbinary-3.1.6.tgz /tmp/igbinary-3.1.6.tgz

# php ext
RUN set -ex \
    && docker-php-ext-install gd pdo_mysql mysqli sockets pcntl imap bcmath \
    && cd /tmp \
    && docker-php-source extract \
    # redis
    && tar -zxvf redis-5.3.2.tgz \ 
    && mv redis-5.3.2 /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
    && docker-php-ext-enable redis \
    && rm redis-5.3.2.tgz \
    # igbinary
    && tar -zxvf igbinary-3.1.6.tgz \ 
    && mv igbinary-3.1.6 /usr/src/php/ext/igbinary \
    && docker-php-ext-install igbinary \
    && docker-php-ext-enable igbinary \
    && rm igbinary-3.1.6.tgz \
    && docker-php-source delete

# check
RUN set -ex \
    && cd /tmp \
    # 检查 PHP 版本
    && php -v \
    # 检查已安装的模块
    && php -m \
    && echo -e "PHP7.1-FPM Build Completed!"

#COPY ./laravel.ini /usr/local/etc/php/conf.d

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000