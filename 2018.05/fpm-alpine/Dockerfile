# DO NOT EDIT: created by update.sh from Dockerfile-alpine.template
FROM php:7.1-fpm-alpine
LABEL maintainer="Philipp Holzer <admin@philipp.info>"

# entrypoint.sh and cron.sh dependencies
RUN set -ex; \
    apk add --no-cache \
        rsync \
        git \
 # For mail() support
        ssmtp;

# install the PHP extensions we need
# see https://friendi.ca/resources/requirements/
RUN set -ex; \
    \
    apk add -U --no-cache --virtual .build-deps \
        libxml2-dev \
        mysql-client \
        bash \
        autoconf \
        g++ \
        make \
        openssl \
        openssl-dev \
        libpng \
        libpng-dev \
        libjpeg-turbo-dev \
        imagemagick-dev \
        imagemagick \
        libtool \
        libmcrypt \
        libmcrypt-dev \
        freetype \
        libpng \
        libjpeg-turbo-dev \
        freetype-dev \
        librsvg \
        libcurl \
        curl \
        curl-dev \
        rsync \
        bzip2 \
        pcre-dev \
    ; \
    pecl install imagick-3.4.3; \
        docker-php-ext-enable imagick; \
        pecl clear-cache; \
    docker-php-ext-configure gd \
        --with-gd \
        --enable-gd-native-ttf \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
    ; \
    docker-php-ext-install -j 4 curl pdo pdo_mysql xml gd zip opcache mbstring posix ctype json iconv mcrypt; \
    \
    runDeps="$( \
      scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
          | tr ',' '\n' \
          | sort -u \
          | awk 'system("[ -e /usr/local/lib" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --virtual .friendica-phpext-rundeps $runDeps; \
    apk del .build-deps;

RUN chown -R www-data:root /var/www; \
    chmod -R g=u /var/www

VOLUME /var/www/html


RUN {\
      echo sendmail_path = "/usr/sbin/sendmail -t -i" ;\
    } > /usr/local/etc/php/conf.d/sendmail.ini;

ENV AUTOINSTALL false
ENV VALIDATION true
ENV SITENAME "Friendica Social Network"

ENV FRIENDICA_VERSION 2018.05
ENV FRIENDICA_ADDONS 2018.05

COPY bin/* /usr/local/bin/
COPY config/* /usr/src/config/
COPY *.sh /
RUN chmod +x /*.sh
RUN chmod +x /usr/local/bin/*

RUN set -ex; \
    curl -fsSL -o friendica.tar.gz \
        "https://github.com/friendica/friendica/archive/${FRIENDICA_VERSION}.tar.gz"; \
    tar -xzf friendica.tar.gz -C /usr/src/; \
    rm friendica.tar.gz; \
    mv -f /usr/src/friendica-${FRIENDICA_VERSION}/ /usr/src/friendica; \
    chmod 777 /usr/src/friendica/view/smarty3; \
    curl -fsSL -o friendica_addons.tar.gz \
        "https://github.com/friendica/friendica-addons/archive/${FRIENDICA_ADDONS}.tar.gz"; \
    mkdir /usr/src/friendica/addon; \
    tar -xzf friendica_addons.tar.gz -C /usr/src/friendica/addon --strip-components=1; \
    rm friendica_addons.tar.gz; \
    friendica composer install;

ENTRYPOINT ["/entrypoint.sh"]
CMD ["php-fpm"]