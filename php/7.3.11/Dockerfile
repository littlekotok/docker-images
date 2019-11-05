FROM debian:buster-slim

# docker build -t kotok:php7.3-fpm .

# https://github.com/docker-library/php/blob/master/7.3/buster/fpm/Dockerfile

# ENV LC_ALL UTF-8
ENV DEBIAN_FRONTEND noninteractive 

ENV PHP_VERSION 7.3.11
ENV MONGODB_VERSION 1.6.0
ENV MEMCACHED_VERSION 3.1.4

ENV PHP_INI_SCAN_DIR "/etc/php${PHP_VERSION}/conf.d"

ENV PHPIZE_DEPS \
        autoconf \
        dpkg-dev \
        file \
        g++ \
        gcc \
        libc-dev \
        make \
        pkg-config \
        re2c

RUN set -eux; \
        apt-get update; \
        apt-get -y --no-install-recommends install \
            nano \
            openssl \
            wget \
            $PHPIZE_DEPS \
            ca-certificates \
            curl \
            xz-utils \
            libreadline-dev \
            libargon2-dev \
            libcurl4-openssl-dev \
            libedit-dev \
            libsodium-dev \
            libsqlite3-dev \
            libssl-dev \
            libxml2-dev \
            libjpeg-dev \
            libzip-dev \
            libpng-dev \
            zlib1g-dev \
            libmemcached-dev \
            build-essential \
            libtool \
	    ;

# Fetch php source
RUN set -eux; \
        cd /opt; \
        curl -O "https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz"; \
        tar -zxf "php-${PHP_VERSION}.tar.gz"; \
        rm -f "php-${PHP_VERSION}.tar.gz";

# Configure & install php
# https://www.php.net/manual/fr/install.unix.debian.php
RUN set -eux; \
        cd "/opt/php-${PHP_VERSION}/"; \
        gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
        ./configure \
            --build="$gnuArch" \
            --prefix="/etc/php${PHP_VERSION}/" \
            # --exec-prefix="/usr" \
            --with-config-file-scan-dir="$PHP_INI_SCAN_DIR" \
            --enable-mysqlnd \
            --with-pdo-mysql \
            --with-pdo-mysql=mysqlnd \
            --enable-intl \
            --enable-bcmath \
            --enable-fpm \
            --with-fpm-user=www-data \
            --with-fpm-group=www-data \
            --enable-mbstring \
            --enable-phpdbg \
            --enable-shmop \
            --enable-sockets \
            --enable-sysvmsg \
            --enable-sysvsem \
            --enable-sysvshm \
            --with-gd \
            --enable-zip \
            --with-libzip="/usr/lib/${gnuArch}" \
            --with-zlib \
            --with-curl \
            --with-pear \
            --with-openssl \
            --enable-pcntl \
            --with-readline \
            --with-libmemcached \
        ; \
        make -j "$(nproc)"; \
        make install; \
        make clean;

# Set php config & binary files
RUN set -eux; \
        cp "/opt/php-${PHP_VERSION}/php.ini-development" "/etc/php${PHP_VERSION}/lib/php.ini"; \
        cp "/opt/php-${PHP_VERSION}/php.ini-production" "/etc/php${PHP_VERSION}/lib/php.ini-production"; \
        ln -s "/etc/php${PHP_VERSION}/bin/php" /usr/bin/php; \
        ln -s "/etc/php${PHP_VERSION}/bin/pecl" /usr/bin/pecl; \
        ln -s "/etc/php${PHP_VERSION}/sbin/php-fpm" /usr/sbin/php-fpm;

# Set php-fpm config files
RUN set -eux; \
        mv "/etc/php${PHP_VERSION}/etc/php-fpm.conf.default" "/etc/php${PHP_VERSION}/etc/php-fpm.conf"; \
        mv "/etc/php${PHP_VERSION}/etc/php-fpm.d/www.conf.default" "/etc/php${PHP_VERSION}/etc/php-fpm.d/www.conf";

# Install extensions with PECL
RUN set -eux; \
        pecl channel-update "pecl.php.net"; \
        pecl install "mongodb-${MONGODB_VERSION}"; \
        pecl install "memcached-${MEMCACHED_VERSION}";

# Load extensions
RUN set -eux; \
        if [ ! -d "$PHP_INI_SCAN_DIR" ]; then \
                mkdir -p "$PHP_INI_SCAN_DIR"; \
        fi; \
        echo "extension=mongodb.so" > "${PHP_INI_SCAN_DIR}/20-mongodb.ini"; \
        echo "extension=memcached.so" > "${PHP_INI_SCAN_DIR}/20-memcached.ini";

# https://www.php.net/manual/fr/install.fpm.php
# change pid on php-fpm.conf
# cp /opt/php-7.3.11/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
# chmod 755 /etc/init.d/php-fpm
# /etc/init.d/php-fpm start
