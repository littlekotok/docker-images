# FROM debian:buster-slim
FROM kotok:php7.3-fpm

# docker build -t kotok:nginx-1.17.5 .
# https://github.com/nginxinc/docker-nginx/blob/fe97d699daae7e04f916771ac520f7cf25ab2b27/mainline/buster/Dockerfile
# https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_debian

ENV DEBIAN_FRONTEND noninteractive

ENV NGINX_VERSION   1.17.5
ENV NJS_VERSION     0.3.6
ENV PKG_RELEASE     1~buster

RUN set -eux; \
	apt-get update; \
	apt-get -y --no-install-recommends install \
	    nano \
	    procps \
	    curl \
	    ; \
	apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates;

# Download the key used to sign NGINX packages and the repository, and add it to the apt program’s key ring:
RUN set -eux; \
	cd /opt; \
	curl -O https://nginx.org/keys/nginx_signing.key; \
	apt-key add nginx_signing.key; \
	echo "deb https://nginx.org/packages/mainline/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list;

RUN set -eux; \
    nginxPackages=" \
		nginx=${NGINX_VERSION}-${PKG_RELEASE} \
		nginx-module-xslt=${NGINX_VERSION}-${PKG_RELEASE} \
		nginx-module-geoip=${NGINX_VERSION}-${PKG_RELEASE} \
		nginx-module-image-filter=${NGINX_VERSION}-${PKG_RELEASE} \
		nginx-module-njs=${NGINX_VERSION}.${NJS_VERSION}-${PKG_RELEASE} \
    " \
    && apt-get update; \
    apt-get install --no-install-recommends --no-install-suggests -y \
	$nginxPackages \
	gettext-base \
    ;
    # apt-get remove --purge --auto-remove -y ca-certificates; \
    # rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list; \

RUN set -eux; \
    if [ -f "/usr/local/bin/docker-entrypoint" ]; then \
		sed -i 's@#HOOK_END@/etc/init.d/nginx start\n#HOOK_END@g' /usr/local/bin/docker-entrypoint; \
    fi;

# CMD ["/etc/init.d/nginx", "start;"]