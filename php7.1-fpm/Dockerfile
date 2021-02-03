FROM php:7.1-fpm

RUN set -ex \
    && cd /tmp \
    # 换源
    && sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    && sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list
    # && apk add --no-cache vim git autoconf openssl-dev build-base zlib-dev re2c libpng-dev oniguruma-dev imap-dev libzip-dev libxslt-dev

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=PRC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

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

# wkhtmltopdf
ARG INSTALL_WKHTMLTOPDF=false
RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
    apt-get install -y -f wkhtmltopdf \
;fi

COPY src/* /tmp/

# php ext
RUN set -ex \
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
    && tar -zxvf swoole-4.3.6.tgz \
    && mv swoole-4.3.6 /usr/src/php/ext/swoole \
    && docker-php-ext-configure swoole --enable-openssl --enable-sockets --enable-http2 --enable-mysqlnd \
    && docker-php-ext-install swoole \
    && rm swoole-4.3.6.tgz \
    # xdebug
    && wget -O xdebug-2.9.8.tar.gz https://github.com/xdebug/xdebug/archive/2.9.8.tar.gz \
    && tar -zxvf xdebug-2.9.8.tar.gz \
    && mv xdebug-2.9.8 /usr/src/php/ext/xdebug \
    && docker-php-ext-install xdebug \
    && rm xdebug-2.9.8.tar.gz \
    # amqp  
    && tar -zxvf amqp-1.10.2.tgz \
    && mv amqp-1.10.2 /usr/src/php/ext/amqp \
    && docker-php-ext-install amqp \
    && rm amqp-1.10.2.tgz \
    # mailparse  
    && tar -zxvf mailparse-3.1.1.tgz \
    && mv mailparse-3.1.1 /usr/src/php/ext/mailparse \
    && docker-php-ext-install mailparse \
    && rm mailparse-3.1.1.tgz \
    # source delete
    && docker-php-source delete
    ## install by pecl
    #&& pecl install xdebug-2.9.8 \
    #&& docker-php-ext-enable xdebug \
    # amqp
    #&& pecl install amqp \
    #&& docker-php-ext-enable amqp \
    # mailparse
    #&& pecl install mailparse \
    #&& docker-php-ext-enable mailparse

# Composer
RUN set -ex \
    && cd /tmp \
    #&& wget https://mirrors.aliyun.com/composer/composer.phar \
    # b2c code can work for 1.9.0
    #&& wget https://getcomposer.org/download/1.9.0/composer.phar \
    && mv composer-1.9.0.phar /usr/local/bin/composer \
    && chmod u+x /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer \
    && echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> ~/.bashrc
    
# Oh My ZSH!
RUN set -ex \
    && wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh \
    && sh install.sh \
    && rm install.sh \
    && sed -i 's/^ZSH_THEME="robbyrussell"/ZSH_THEME="ys"/' ~/.zshrc

#COPY ./laravel.ini /usr/local/etc/php/conf.d

# Check
RUN set -ex \
    && cd /tmp \
    && php -v \
    && php -m \
    && echo -e "PHP7.1-FPM Build Completed!"

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000