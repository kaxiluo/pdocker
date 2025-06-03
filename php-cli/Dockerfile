ARG DOCKER_REPOSITORY
ARG PHP_VERSION
ARG FPM_OR_CLI=cli
ARG DEBIAN_RELEASE

ARG DEBIAN_RELEASE_TAG=${DEBIAN_RELEASE:+-${DEBIAN_RELEASE}}

FROM ${DOCKER_REPOSITORY}/php:${PHP_VERSION}-${FPM_OR_CLI}${DEBIAN_RELEASE_TAG}

RUN set -xe \
    && apt-get update -yqq \
    && apt-get install -y openssh-client

ARG INSTALL_PGSQL=false
RUN if [ ${INSTALL_PGSQL} = true ]; then \
    docker-php-ext-install pgsql \
;fi

COPY ./myfile/.ssh/id_ed25519 /root/.ssh/id_ed25519    
COPY ./myfile/.ssh/id_ed25519.pub /root/.ssh/id_ed25519.pub

WORKDIR /var/www
