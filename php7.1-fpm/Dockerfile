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

COPY src/redis-5.3.2.tgz /tmp/redis-5.3.2.tgz
COPY src/igbinary-3.1.6.tgz /tmp/igbinary-3.1.6.tgz

# php ext
RUN set -ex \
    && pecl channel-update pecl.php.net \
    && docker-php-ext-install gd pdo_mysql mysqli exif zip sockets pcntl imap bcmath \
    && cd /tmp \
    && docker-php-source extract \
    # redis
    && tar -zxvf redis-5.3.2.tgz \ 
    && mv redis-5.3.2 /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
    && rm redis-5.3.2.tgz \
    # igbinary
    && tar -zxvf igbinary-3.1.6.tgz \ 
    && mv igbinary-3.1.6 /usr/src/php/ext/igbinary \
    && docker-php-ext-install igbinary \
    && rm igbinary-3.1.6.tgz \
    && docker-php-source delete \
    # xdebug
    && pecl install xdebug-2.9.8 \
    && docker-php-ext-enable xdebug

RUN set -ex \ 
    # 下载composer 
    #&& wget https://mirrors.aliyun.com/composer/composer.phar \
    # B2C的代码使用最新版本安装报错，所以下载1.9.0
    && wget https://getcomposer.org/download/1.9.0/composer.phar \
    && chmod u+x composer.phar \
    && mv composer.phar /usr/local/bin/composer \
    # 给 composer 设置aliyun镜像
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer \
    # 把 composer 全局命令加入 PATH
    && echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> ~/.bashrc

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