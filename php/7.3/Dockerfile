FROM debian:buster-slim

# docker build -t kotok:php7.3-fpm .

# https://github.com/docker-library/php/blob/master/7.3/buster/fpm/Dockerfile

# ENV LC_ALL UTF-8
ENV DEBIAN_FRONTEND noninteractive

ENV PHP_MAJOR 7.3
ENV PHP_VERSION 7.3.12
ENV MONGODB_VERSION 1.6.1
ENV MEMCACHED_VERSION 3.1.5

ENV PHP_INI_SCAN_DIR "/etc/php${PHP_MAJOR}/conf.d"

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
            procps \
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
            --prefix="/etc/php${PHP_MAJOR}/" \
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
        cp "/opt/php-${PHP_VERSION}/php.ini-development" "/etc/php${PHP_MAJOR}/lib/php.ini"; \
        cp "/opt/php-${PHP_VERSION}/php.ini-development" "/etc/php${PHP_MAJOR}/etc/php.ini"; \
        ln -s "/etc/php${PHP_MAJOR}/bin/php" /usr/bin/php; \
        ln -s "/etc/php${PHP_MAJOR}/bin/pecl" /usr/bin/pecl; \
        ln -s "/etc/php${PHP_MAJOR}/sbin/php-fpm" /usr/sbin/php-fpm; \
        php --version;

# Set php-fpm config files
RUN set -eux; \
        mv "/etc/php${PHP_MAJOR}/etc/php-fpm.conf.default" "/etc/php${PHP_MAJOR}/etc/php-fpm.conf"; \
        mv "/etc/php${PHP_MAJOR}/etc/php-fpm.d/www.conf.default" "/etc/php${PHP_MAJOR}/etc/php-fpm.d/www.conf";

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
RUN set -eux; \
        cp "/opt/php-${PHP_VERSION}/sapi/fpm/init.d.php-fpm" /etc/init.d/php-fpm; \
        sed -i 's@php_fpm_PID=\${prefix}@php_fpm_INI=\${prefix}/etc/php.ini\nphp_fpm_PID=\${prefix}@g' /etc/init.d/php-fpm; \
        sed -i 's@--pid \$php_fpm_PID@--pid \$php_fpm_PID -c \$php_fpm_INI@g' /etc/init.d/php-fpm; \
        chmod 755 /etc/init.d/php-fpm;

COPY docker-entrypoint /usr/local/bin/

RUN set -eux; \
        chmod 755 /usr/local/bin/docker-entrypoint

ENTRYPOINT ["docker-entrypoint"]
