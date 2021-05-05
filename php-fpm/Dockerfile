ARG X_PHP_VERSION
FROM php:${X_PHP_VERSION}-fpm

ARG X_PHP_VERSION

RUN set -ex \
    && cd /tmp \
    && { \
		echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free"; \
		echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free"; \
		echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free"; \
		echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free"; \
	} > /etc/apt/sources.list ;
    #&& sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    #&& sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    #&& sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=PRC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN set -xe \
    && apt-get update -y \
    && pecl channel-update pecl.php.net \
    && apt-get install -y \
      dialog whiptail \
      apt-utils telnet net-tools procps iputils-ping \
      vim git wget curl zsh

RUN set -eux \
    && apt-get install -y --no-install-recommends \
            autoconf libzip-dev zip unzip \
            libmemcached-dev libz-dev libpq-dev \
            libjpeg-dev libpng-dev  libfreetype6-dev libwebp-dev \
            libssl-dev libmcrypt-dev  libonig-dev \
            libc-client-dev libkrb5-dev \
            libxslt-dev librabbitmq-dev

COPY src/* /tmp/

# wkhtmltopdf
ARG INSTALL_WKHTMLTOPDF=false
RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
    apt-get install -y \
    libxrender1 \
    libfontconfig1 \
    libx11-dev \
    libjpeg62 \
    libxtst6 \
    fontconfig \ 
    libjpeg62-turbo \
    xfonts-base \
    xfonts-75dpi \
    #&& wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.stretch_amd64.deb \
    && dpkg -i /tmp/wkhtmltox_0.12.6-1.stretch_amd64.deb \
    && apt -f install \
;fi

# php ext
RUN set -ex \
    && cd /tmp \
    # pdo_mysql mysqli ......
    && docker-php-ext-install pdo_mysql mysqli exif zip sockets pcntl bcmath xsl gd \
    # imap
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    # source extract
    && docker-php-source extract \
    # redis
    && tar -zxvf redis-5.3.2.tgz \ 
    && mv redis-5.3.2 /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
    # igbinary
    && tar -zxvf igbinary-3.1.6.tgz \ 
    && mv igbinary-3.1.6 /usr/src/php/ext/igbinary \
    && docker-php-ext-install igbinary \
    # amqp  
    && tar -zxvf amqp-1.10.2.tgz \
    && mv amqp-1.10.2 /usr/src/php/ext/amqp \
    && docker-php-ext-install amqp \
    # mailparse  
    && tar -zxvf mailparse-3.1.1.tgz \
    && mv mailparse-3.1.1 /usr/src/php/ext/mailparse \
    && docker-php-ext-install mailparse \
    # mongodb  
    && tar -zxvf mongodb-1.9.1.tgz \
    && mv mongodb-1.9.1 /usr/src/php/ext/mongodb \
    && docker-php-ext-install mongodb

## swoole
ARG INSTALL_SWOOLE=false
ARG SWOOLE_VERSION=4.6.3
RUN if [ ${INSTALL_SWOOLE} = true ]; then \
    cd /tmp && \
    tar -zxvf swoole-${SWOOLE_VERSION}.tgz && \
    mv swoole-${SWOOLE_VERSION} /usr/src/php/ext/swoole \
    && docker-php-ext-configure swoole --enable-openssl --enable-sockets --enable-http2 --enable-mysqlnd \
    && docker-php-ext-install swoole \
;fi

## xdebug
ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \ 
    cd /tmp \
    #pecl install xdebug-2.9.8 \
    #docker-php-ext-enable xdebug \
    #wget -O xdebug-2.9.8.tar.gz https://github.com/xdebug/xdebug/archive/2.9.8.tar.gz \
    && tar -zxvf xdebug-2.9.8.tgz \
    && mv xdebug-2.9.8 /usr/src/php/ext/xdebug \
    && docker-php-ext-install xdebug \
;fi

## xhprof
ARG INSTALL_XHPROF=false
RUN if [ ${INSTALL_XHPROF} = true ]; then \
    cd /tmp \
    # curl -L -o /tmp/xhprof.tar.gz "https://github.com/tideways/php-xhprof-extension/archive/v5.0.1.tar.gz"; \
    && mkdir -p xhprof \
    && tar -C xhprof -zxvf /tmp/php-xhprof-extension-5.0.4.tar.gz --strip 1 \
    && ( \
        cd xhprof \
        && phpize \
        && ./configure \
        && make \
        && make install \
    ) \
;fi

COPY ./xhprof.ini /usr/local/etc/php/conf.d

RUN if [ ${INSTALL_XHPROF} = false ]; then \
    rm /usr/local/etc/php/conf.d/xhprof.ini \
;fi

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
ARG INSTALL_ZSH=false
RUN if [ ${INSTALL_ZSH} = true ]; then \ 
    set -ex \
    #&& wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh \
    #&& sh install.sh \
    #&& rm install.sh \
    && cd /tmp \
    && sh oh-my-zsh-install.sh \
    && rm oh-my-zsh-install.sh \
    && sed -i 's/^ZSH_THEME="robbyrussell"/ZSH_THEME="ys"/' ~/.zshrc \
;fi

#COPY ./laravel.ini /usr/local/etc/php/conf.d

# Check
RUN set -ex \
    && cd /tmp \
    && php -v \
    && php -m \
    && echo -e "PHP${PHP_VERSION}-FPM Build Completed!"

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog && \
    docker-php-source delete

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000