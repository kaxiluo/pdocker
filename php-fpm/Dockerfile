ARG X_PHP_VERSION
FROM linyiyong/php-fpm:${X_PHP_VERSION}

ARG X_PHP_VERSION

ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
    cd /tmp \
    && { \
        echo "deb http://mirrors.aliyun.com/debian/ buster main non-free contrib"; \
        echo "deb-src http://mirrors.cloud.aliyuncs.com/debian/ buster main non-free contrib"; \
        echo "deb http://mirrors.cloud.aliyuncs.com/debian-security buster/updates main"; \
        echo "deb-src http://mirrors.cloud.aliyuncs.com/debian-security buster/updates main"; \
        echo "deb http://mirrors.cloud.aliyuncs.com/debian/ buster-updates main non-free contrib"; \
        echo "deb-src http://mirrors.cloud.aliyuncs.com/debian/ buster-updates main non-free contrib"; \
        echo "deb http://mirrors.cloud.aliyuncs.com/debian/ buster-backports main non-free contrib"; \
        echo "deb-src http://mirrors.cloud.aliyuncs.com/debian/ buster-backports main non-free contrib"; \
	} > /etc/apt/sources.list \
    #&& sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    #&& sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list \
    #&& sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list
;fi

COPY src/* /tmp/

# php ext
RUN set -ex \
    && cd /tmp \
    && docker-php-ext-install opcache

# wkhtmltopdf
ARG INSTALL_WKHTMLTOPDF=false
RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
    apt-get update \
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

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    #rm /var/log/lastlog /var/log/faillog && \
    docker-php-source delete

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000