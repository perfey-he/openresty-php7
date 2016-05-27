FROM debian:jessie

RUN apt-get update \
 && apt-get install -y vim \
 && apt-get install -y --no-install-recommends \
    curl perl make build-essential procps \
    libreadline-dev libncurses5-dev libpcre3-dev libssl-dev \
 && apt-get install -y unzip \
 && rm -rf /var/lib/apt/lists/* \
 && alias ll='ls -althr --color'

ENV OPENRESTY_VERSION 1.9.7.4
ENV OPENRESTY_PREFIX /etc/openresty
ENV NGINX_PREFIX /etc/openresty/nginx
ENV VAR_PREFIX /run/nginx

# NginX prefix is automatically set by OpenResty to $OPENRESTY_PREFIX/nginx
# look for $ngx_prefix in https://github.com/openresty/ngx_openresty/blob/master/util/configure

RUN cd /root \
 && echo "==> Downloading OpenResty..." \
 && curl -sSL http://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz | tar -xvz \
 && echo "==> Configuring OpenResty..." 

RUN cd /root \
 && curl -sSLk https://github.com/maxmind/libmaxminddb/releases/download/1.1.1/libmaxminddb-1.1.1.tar.gz -o libmaxminddb.tar.gz \
 && mkdir openresty-${OPENRESTY_VERSION}/libmaxminddb \
 && tar -xzvf libmaxminddb.tar.gz  -C openresty-*/libmaxminddb --strip-components=1 \
 && cd /root/openresty-*/libmaxminddb \
 && ./configure \
 && make \
 && make check \ 
 && make install \
 && ldconfig

#RUN cd /root \
# && curl -sSLk http://people.freebsd.org/~osa/ngx_http_redis-0.3.7.tar.gz -o ngx-http-redis.tar.gz \
# && mkdir openresty-${OPENRESTY_VERSION}/http-redis \
# && tar -xzvf ngx-http-redis.tar.gz -C /root/openresty-*/http-redis --strip-components=1

#RUN cd /root \ 
# && curl -sSLk https://github.com/leev/ngx_http_geoip2_module/archive/1.0.tar.gz -o ngx-http-geoip2.tar.gz \
# && mkdir openresty-${OPENRESTY_VERSION}/http-geoip2 \
# && tar -xzvf ngx-http-geoip2.tar.gz  -C /root/openresty-*/http-geoip2 --strip-components=1 

RUN cd /root \
 && curl -sSL http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz -o ngx_cache_purge-2.3.tar.gz \
 && mkdir openresty-${OPENRESTY_VERSION}/ngx_cache_purge \
 && tar -xzvf ngx_cache_purge-2.3.tar.gz  -C /root/openresty-*/ngx_cache_purge  --strip-components=1
 
#RUN cd /root \
# && curl -sSLk https://github.com/simpl/ngx_devel_kit/archive/v0.2.19.zip -o ngx_devel_kit-0.2.19.zip \
# && unzip ngx_devel_kit-0.2.19.zip -d /root/openresty-*/ \
# && mv /root/openresty-${OPENRESTY_VERSION}/ngx_devel_kit-0.2.19 /root/openresty-${OPENRESTY_VERSION}/ngx_devel_kit

RUN cd /root \
 && curl -sSLk https://github.com/p0pr0ck5/lua-resty-waf/archive/master.zip -o lua-resty-waf-master.zip \
 && unzip lua-resty-waf-master.zip -d /root/openresty-*/bundle/

RUN cd /root/openresty-* \
 && readonly NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
 && echo "using upto $NPROC threads" \
 && ./configure \
    --prefix=$OPENRESTY_PREFIX \
    --http-client-body-temp-path=$VAR_PREFIX/client_body_temp \
    --http-proxy-temp-path=$VAR_PREFIX/proxy_temp \
    --http-log-path=/var/log/access.log \
    --error-log-path=var/log/error.log \
    --pid-path=$VAR_PREFIX/nginx.pid \
    --lock-path=$VAR_PREFIX/nginx.lock \
    --with-luajit \
    --with-pcre-jit \
    --with-ipv6 \
    --with-http_ssl_module \
    --with-http_gzip_static_module \
    --with-http_stub_status_module \
#    --with-http_spdy_module \
#    --with-http_image_filter_module \
    --with-file-aio \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_realip_module \
    --with-http_auth_request_module \
    --with-http_addition_module \
    --with-http_sub_module \
#    --add-module=http-redis \
#    --add-module=http-geoip2 \
#    --add-module=./ngx_devel_kit \
    --add-module=./ngx_cache_purge \
    --without-http_ssi_module \
    --without-http_userid_module \
    --without-http_uwsgi_module \
    --without-http_scgi_module \
    -j${NPROC} \
 && echo "==> Building OpenResty..." \
 && make -j${NPROC} \
 && echo "==> Installing OpenResty..." \
 && make install \
 && echo "==> Finishing..." \
 && ln -sf $NGINX_PREFIX/sbin/nginx /usr/local/bin/nginx \
 && ln -sf $NGINX_PREFIX/sbin/nginx /usr/local/bin/openresty \
 && ln -sf $OPENRESTY_PREFIX/bin/resty /usr/local/bin/resty \
 && ln -sf $OPENRESTY_PREFIX/luajit/bin/luajit-* $OPENRESTY_PREFIX/luajit/bin/lua \
 && ln -sf $OPENRESTY_PREFIX/luajit/bin/luajit-* /usr/local/bin/lua \
 && rm -rf /root/ngx_openresty*

WORKDIR $NGINX_PREFIX/
RUN mkdir -p /var/log/nginx /data/www 
RUN cp -r /root/openresty-${OPENRESTY_VERSION}/bundle/lua-resty-waf-master /etc/openresty/lualib/lua-resty-waf-master

VOLUME /var/log/nginx

########################################################################################################################################

ENV PHPIZE_DEPS \
		autoconf \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkg-config \
		re2c
RUN apt-get update && apt-get install -y \
		$PHPIZE_DEPS \
		ca-certificates \
		curl \
		libedit2 \
		libsqlite3-0 \
		libxml2 \
	--no-install-recommends && rm -r /var/lib/apt/lists/*

ENV PHP_INI_DIR /usr/local/etc/php
RUN mkdir -p $PHP_INI_DIR/conf.d

##<autogenerated>##
ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data
##</autogenerated>##

ENV GPG_KEYS 1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763

ENV PHP_VERSION 7.0.6
ENV PHP_FILENAME php-7.0.6.tar.xz
ENV PHP_SHA256 1b237a9455e5476a425dbb9d99966bad68107747c601958cb9558a7fb49ab419

RUN set -xe \
	&& buildDeps=" \
		$PHP_EXTRA_BUILD_DEPS \
		libcurl4-openssl-dev \
		libedit-dev \
		libsqlite3-dev \
		libssl-dev \
		libxml2-dev \
		xz-utils \
	" \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends && rm -rf /var/lib/apt/lists/* \
	&& curl -fSL "http://php.net/get/$PHP_FILENAME/from/this/mirror" -o "$PHP_FILENAME" \
	&& echo "$PHP_SHA256 *$PHP_FILENAME" | sha256sum -c - \
	&& curl -fSL "http://php.net/get/$PHP_FILENAME.asc/from/this/mirror" -o "$PHP_FILENAME.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& for key in $GPG_KEYS; do \
		gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key"; \
	done \
	&& gpg --batch --verify "$PHP_FILENAME.asc" "$PHP_FILENAME" \
	&& rm -r "$GNUPGHOME" "$PHP_FILENAME.asc" \
	&& mkdir -p /usr/src/php \
	&& tar -xf "$PHP_FILENAME" -C /usr/src/php --strip-components=1 \
	&& rm "$PHP_FILENAME" \
	&& cd /usr/src/php \
	&& ./configure \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		$PHP_EXTRA_CONFIGURE_ARGS \
		--disable-cgi \
# --enable-mysqlnd is included here because it's harder to compile after the fact than extensions are (since it's a plugin for several extensions, not an extension in itself)
		--enable-mysqlnd \
# --enable-mbstring is included here because otherwise there's no way to get pecl to use it properly (see https://github.com/docker-library/php/issues/195)
		--enable-mbstring \
		--with-curl \
		--with-libedit \
		--with-openssl \
		--with-zlib \
	&& make -j"$(nproc)" \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } \
	&& make clean \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps

COPY docker-php-ext-* /usr/local/bin/

##<autogenerated>##
WORKDIR /var/www/html

RUN set -ex \
	&& cd /usr/local/etc \
	&& if [ -d php-fpm.d ]; then \
		# for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
		sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
		cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
		# PHP 5.x don't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
		mkdir php-fpm.d; \
		cp php-fpm.conf.default php-fpm.d/www.conf; \
		{ \
			echo '[global]'; \
			echo 'include=etc/php-fpm.d/*.conf'; \
		} | tee php-fpm.conf; \
	fi \
	&& { \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
	} | tee php-fpm.d/docker.conf \
	&& { \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = [::]:9000'; \
	} | tee php-fpm.d/zz-docker.conf

RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        git \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

RUN apt-get install -y wget unzip zlib1g-dev

#redis.so memcached.so will install at /usr/local/lib/php/extensions/no-debug-non-zts-20151012/
RUN cd /root/ \
    && wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz \
    && tar zxvf libmemcached-1.0.18.tar.gz \
    && cd libmemcached-1.0.18 \
    && ./configure \
    && make -j"$(nproc)" \
    && make install

RUN cd /root/ \
    && wget -c https://github.com/phpredis/phpredis/archive/php7.zip \
    && mv php7.zip php7-redis.zip \
    && unzip php7-redis.zip \
    && cd phpredis-php7/ \
    && /usr/local/bin/phpize \
    && ./configure --with-php-config=/usr/local/bin/php-config \
    && make -j"$(nproc)" \
    && make install

RUN cd /root/ \
    && wget https://codeload.github.com/php-memcached-dev/php-memcached/zip/php7 \
    && mv php7  php-memcached-php7.zip \
    && unzip php-memcached-php7.zip \
    && cd php-memcached-php7 \
    && /usr/local/bin/phpize \
    && ./configure --with-php-config=/usr/local/bin/php-config   --disable-memcached-sasl \
    && make -j"$(nproc)" \
    && make install

COPY php.ini /usr/local/etc/php

#ONBUILD RUN rm -rf conf/* /data/nginx/*
#ONBUILD COPY nginx $NGINX_PREFIX/

COPY startup /root

EXPOSE 80 9000
CMD [/root/startup]
#CMD ["php-fpm"]
#CMD ["nginx", "-g", "daemon off;"]
