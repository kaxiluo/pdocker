ARG X_PHP_VERSION
FROM linyiyong/php-cli:${X_PHP_VERSION}

ARG X_PHP_VERSION

ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    cd /tmp \
    && cp /etc/apt/sources.list /etc/apt/sources.list.bak \
    && sed -i -E 's/(deb|security).debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list \
;fi

# update
RUN set -xe \
    && apt-get update -y \
    && pecl channel-update pecl.php.net \
    && apt-get install -y --no-install-recommends

# only use
# php-xhprof-extension-5.0.4.tar.g
# wkhtmltox_0.12.6-1.stretch_amd64.deb
COPY src/* /tmp/

#######################
# PHP EXT 可选
#######################

RUN docker-php-source extract

# IMAGEMAGICK
ARG INSTALL_IMAGEMAGICK=false
ARG IMAGEMAGICK_VERSION=latest
ENV IMAGEMAGICK_VERSION ${IMAGEMAGICK_VERSION}
RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get update -y  && \
    apt-get install -yqq libmagickwand-dev libmagickcore-dev imagemagick && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
      cd /tmp && \
      if [ ${IMAGEMAGICK_VERSION} = "latest" ]; then \
        git clone https://github.com/Imagick/imagick; \
      else \
        git clone --branch ${IMAGEMAGICK_VERSION} https://github.com/Imagick/imagick; \
      fi && \
      cd imagick && \
      phpize && \
      ./configure && \
      make && \
      make install && \
      rm -r /tmp/imagick; \
    else \
      pecl install imagick; \
    fi && \
    docker-php-ext-enable imagick; \
    php -m | grep -q 'imagick' \
;fi
# imagick
# ARG INSTALL_IMAGEMAGICK=false
# RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
#     #pecl install imagick; \
#     #docker-php-ext-enable imagick \
#     cd /tmp && \
#     apt-get install libmagickcore-dev libmagickwand-dev && \
#     tar -zxvf imagick-3.5.1.tgz && \
#     mv imagick-3.5.1 /usr/src/php/ext/imagick \
#     && docker-php-ext-install imagick \
# ;fi

## swoole
ARG INSTALL_SWOOLE=false
RUN set -eux; \
    if [ ${INSTALL_SWOOLE} = true ]; then \
      # Install Php Swoole Extension
      if   [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "50600" ]; then \
        pecl install swoole-2.0.10; \
      elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70000" ]; then \
        pecl install swoole-4.3.5; \
      elif [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70100" ]; then \
        pecl install swoole-4.5.11; \
      elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
        pecl install -D 'enable-openssl="yes"' swoole-4.8.12; \
      else \
        pecl install -D 'enable-openssl="yes"' swoole; \
      fi; \
      docker-php-ext-enable swoole; \
      php -m | grep -q 'swoole'; \
    fi

# xdebug
ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
  # Install the xdebug extension
  # https://xdebug.org/docs/compat
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ] || { [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && { [ $(php -r "echo PHP_MINOR_VERSION;") = "4" ] || [ $(php -r "echo PHP_MINOR_VERSION;") = "3" ] ;} ;}; then \
    if [ ${LARADOCK_PHP_VERSION} = "8.2" ]; then \
      pecl install xdebug-3.2.0; \
    else \
      pecl install xdebug-3.1.4; \
    fi; \
  else \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install xdebug-2.5.5; \
    else \
      if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
        pecl install xdebug-2.9.0; \
      else \
        pecl install xdebug-2.9.8; \
      fi \
    fi \
  fi && \
  docker-php-ext-enable xdebug \
;fi

## xhprof
ARG INSTALL_XHPROF=false
COPY ./xhprof.ini /usr/local/etc/php/conf.d
RUN if [ ${INSTALL_XHPROF} = false ]; then \
    rm /usr/local/etc/php/conf.d/xhprof.ini \
;fi
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


###################
## Other
###################

# wkhtmltopdf
ARG INSTALL_WKHTMLTOPDF=false
RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
    apt-get update -y \
    && apt-get install -y \
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

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

COPY ./php-common.ini /usr/local/etc/php/conf.d

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    #rm /var/log/lastlog /var/log/faillog && \
    docker-php-source delete

WORKDIR /var/www
