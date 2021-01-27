FROM php:7.4.14-fpm

RUN set -ex \
    && cd /tmp \
    # 换源
    && sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    && sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list

ENV DEBIAN_FRONTEND noninteractive

RUN set -xe \
    && apt-get update -y \
    && pecl channel-update pecl.php.net \
    && apt-get install -y \
      dialog whiptail \
      apt-utils procps iputils-ping vim git wget curl zsh

RUN set -eux \
    && apt-get install -y --no-install-recommends \
            autoconf libzip-dev zip unzip \
            libmemcached-dev libz-dev libpq-dev \
            libjpeg-dev libpng-dev  libfreetype6-dev libwebp-dev \
            libssl-dev libmcrypt-dev  libonig-dev \
            libc-client-dev libkrb5-dev \
            libxslt-dev librabbitmq-dev

RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
    && apt-get install -y -f wkhtmltopdf \
;fi

RUN set -ex \ 
    && rm -rf /var/lib/apt/lists/*

COPY src/swoole-4.4.19.tgz /tmp/swoole-4.4.19.tgz
COPY src/redis-5.3.2.tgz /tmp/redis-5.3.2.tgz
COPY src/igbinary-3.1.6.tgz /tmp/igbinary-3.1.6.tgz

# php ext
RUN set -ex \
    # pdo_mysql mysqli ....
    && docker-php-ext-install pdo_mysql mysqli exif zip sockets pcntl bcmath xsl \
    # gd
    && docker-php-ext-configure gd --prefix=/usr --with-jpeg --with-webp --with-freetype \
    && docker-php-ext-install gd \
    && php -r 'var_dump(gd_info());' \
    # imap
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    # source extract
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
    # swoole
    && tar -zxvf swoole-4.4.19.tgz \
    && mv swoole-4.4.19 /usr/src/php/ext/swoole \
    && docker-php-ext-configure swoole --enable-openssl --enable-sockets --enable-http2 --enable-mysqlnd \
    && docker-php-ext-install swoole \
    && rm swoole-4.4.19.tgz \
    && docker-php-source delete \
    ## install by pecl
    # amqp
    && pecl install amqp \
    && docker-php-ext-enable amqp \
    # mailparse
    && pecl install mailparse \
    && docker-php-ext-enable mailparse

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

# Oh My ZSH!
RUN set -ex \
    && wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh \
    && sh install.sh \
    && rm install.sh \
    && sed -i 's/^ZSH_THEME="robbyrussell"/ZSH_THEME="ys"/' ~/.zshrc

# check
RUN set -ex \
    && cd /tmp \
    && php -v \
    && php -m \
    && echo -e "PHP7.4-FPM Build Completed!"

#COPY ./laravel.ini /usr/local/etc/php/conf.d

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000