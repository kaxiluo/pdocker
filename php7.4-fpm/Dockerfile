FROM php:7.4.13-fpm-alpine3.12

# apk 是 alpine 的一个包管理器
RUN set -ex \
    # 在临时目录进行这一切
    && cd /tmp \
    # 把 apk 的默认源改为aliyun镜像
    && sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    # 更新包列表
    && apk update \
    # 添加这么多扩展是因为后面我们编译 swoole imap 需要用到 
    && apk add --no-cache vim git autoconf openssl-dev build-base zlib-dev re2c libpng-dev oniguruma-dev imap-dev

COPY ext/swoole-4.4.19.tgz /tmp/swoole-4.4.19.tgz
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
    # swoole
    && tar -zxvf swoole-4.4.19.tgz \
    && mv swoole-4.4.19 /usr/src/php/ext/swoole \
    && docker-php-ext-configure swoole --enable-openssl --enable-sockets --enable-http2 --enable-mysqlnd \
    && docker-php-ext-install swoole \
    && docker-php-ext-enable swoole \
    && rm swoole-4.4.19.tgz \
    && docker-php-source delete

# check
RUN set -ex \
    && cd /tmp \
    # 检查 PHP 版本
    && php -v \
    # 检查已安装的模块
    && php -m \
    && echo -e "Build Completed!"

#COPY ./laravel.ini /usr/local/etc/php/conf.d

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000