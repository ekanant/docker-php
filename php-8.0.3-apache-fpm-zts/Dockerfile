#
# docker build -t ninjabenz/php:8.0.3-apache-fpm-zts .
# docker build --no-cache -t ninjabenz/php:8.0.3-apache-fpm-zts .
#
FROM centos:centos7.9.2009

ENV PHPIZE_DEPS="libmcrypt-devel libicu-devel bzip2-devel curl-devel \
    libxml2-devel bison-devel libzip-devel libaio-devel openssl-devel \
    zlib-devel libpng-devel libjpeg-devel libwebp-devel freetype-devel"
ENV N_PROC=12

RUN yum groupinstall -y "Development tools" \
    && yum install -y wget expat-devel gcc gcc-c++ glibc ${PHPIZE_DEPS} \
    && yum clean all
        
ENV SRC_DIR=/usr/local/src 
ENV LOCAL_D=/usr/local
ENV LD_CONF_D=/etc/ld.so.conf.d

#Install cmake for libzip, brotli
RUN cd ${SRC_DIR} \
    && wget "https://github.com/Kitware/CMake/releases/download/v3.20.0/cmake-3.20.0.tar.gz" \
    && tar -xzvf cmake-3.20.0.tar.gz \
    && cd cmake-3.20.0 \
    && ./configure --prefix="${LOCAL_D}/cmake" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/cmake-* \
    && ln -s ${LOCAL_D}/cmake/bin/cmake ${LOCAL_D}/bin/

#Install apr
RUN cd ${SRC_DIR} \
    && wget "https://www-us.apache.org/dist/apr/apr-1.7.0.tar.gz" \
    && tar -xzvf apr-1.7.0.tar.gz \
    && cd apr-1.7.0 \
    && ./configure --prefix="${LOCAL_D}/apr" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/apr-1.7.0* \
    && echo "${LOCAL_D}/apr/lib" > ${LD_CONF_D}/apr.conf \
    && ldconfig


#Install apr-util
RUN cd ${SRC_DIR} \
    && wget "http://www-eu.apache.org/dist/apr/apr-util-1.6.1.tar.gz" \
    && tar -xzvf apr-util-1.6.1.tar.gz \
    && cd apr-util-1.6.1 \
    && ./configure --prefix="${LOCAL_D}/apr-util" --with-apr="${LOCAL_D}/apr/bin/apr-1-config" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/apr-util-1.6.1* \
    && echo "${LOCAL_D}/apr-util/lib" > ${LD_CONF_D}/apr-util.conf \
    && ldconfig

#Install pcre
RUN cd ${SRC_DIR} \
    && wget "https://ftp.pcre.org/pub/pcre/pcre-8.43.tar.gz" \
    && tar -xzvf pcre-8.43.tar.gz \
    && cd pcre-8.43 \
    && ./configure --prefix="${LOCAL_D}/pcre" --with-apr="${LOCAL_D}/apr/bin/apr-1-config" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/pcre-8.43* \
    && echo "${LOCAL_D}/pcre/lib" > ${LD_CONF_D}/pcre.conf \
    && ldconfig

#Install python3 for nghtp2
RUN cd ${SRC_DIR} \
    && wget "https://www.python.org/ftp/python/3.9.4/Python-3.9.4.tgz" \
    && tar xvf Python-3.9.4.tgz \
    && cd Python-3.9.4 \
    && ./configure --enable-optimizations \ 
    && make altinstall \
    && ln -s /usr/local/bin/python3.9 /usr/local/bin/python3 \
    && rm -f ${SRC_DIR}/Python-3.9.4.tgz \
    && rm -rf ${SRC_DIR}/Python-3.9.4

#Install nghttp2 for apache
RUN cd ${SRC_DIR} \
    && wget "https://github.com/nghttp2/nghttp2/releases/download/v1.43.0/nghttp2-1.43.0.tar.gz" \
    && tar -xzvf nghttp2-1.43.0.tar.gz \
    && cd nghttp2-1.43.0 \
    && ./configure --prefix="${LOCAL_D}/nghttp2" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/nghttp2-1.43.0* \
    && echo "${LOCAL_D}/nghttp2/lib" > ${LD_CONF_D}/nghttp2.conf \
    && ldconfig

#Install brotli
RUN cd ${SRC_DIR} \
    && git clone https://github.com/google/brotli.git \
    && cd brotli/ \
    && mkdir out && cd out \
    && ../configure-cmake --prefix="${LOCAL_D}/brotli" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/brotli* \
    && echo "${LOCAL_D}/brotli/lib" > ${LD_CONF_D}/brotli.conf \
    && ldconfig

#Install apache
ENV APACHE_HOME=${LOCAL_D}/apache2
RUN cd ${SRC_DIR} \
    && wget "https://downloads.apache.org//httpd/httpd-2.4.46.tar.gz" \
    && tar -xzvf httpd-2.4.46.tar.gz \
    && cd httpd-2.4.46 \
    && ./configure --prefix="${APACHE_HOME}" \
        --enable-modules=all --enable-mods-shared=most --enable-mods-static=most --enable-so \
        --enable-deflate --enable-mime-magic --enable-expires --enable-session --enable-ssl \
        --enable-static-rotatelogs --enable-static-logresolve --enable-rewrite --enable-vhost-alias \
        --with-apr="${LOCAL_D}/apr" --with-apr-util="${LOCAL_D}/apr-util" --with-pcre="${LOCAL_D}/pcre" --enable-http2 \
        --with-nghttp2="${LOCAL_D}/nghttp2" --enable-brotli --with-brotli="${LOCAL_D}/brotli" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/httpd* \
    && cd ${LOCAL_D}/bin \ 
    && ln -sf /dev/stdout "${APACHE_HOME}/logs/access_log" \
    && ln -sf /dev/stderr "${APACHE_HOME}/logs/error_log" \
    && ln -s "${APACHE_HOME}/bin/apachectl" /usr/local/bin/ \
    && ln -s "${APACHE_HOME}/bin/httpd" /usr/local/bin/ \
# smoke test
	&& apachectl -v

# https://httpd.apache.org/docs/2.4/stopping.html#gracefulstop
STOPSIGNAL WINCH

EXPOSE 80 443 9000
WORKDIR ${APACHE_HOME}/htdocs

#Install oracle instantclient
ENV ORACLE_INSTANTCLIENT_HOME=${LOCAL_D}/instantclient_21_1
RUN cd ${LOCAL_D} \
    && wget "https://github.com/ekanant/oracle_instantclient/raw/master/21.1.0.0.0/instantclient-basic-linux.x64-21.1.0.0.0.zip" \
    && wget "https://github.com/ekanant/oracle_instantclient/raw/master/21.1.0.0.0/instantclient-sdk-linux.x64-21.1.0.0.0.zip" \
    && unzip instantclient-basic-*.zip \
    && unzip instantclient-sdk-*.zip \
    && rm -rf instantclient-*.zip \ 
    && echo "${ORACLE_INSTANTCLIENT_HOME}" > "${LD_CONF_D}/oracle_instantclient.conf" \
    && ldconfig

#Install libzip
RUN cd ${SRC_DIR} \
    && wget "https://github.com/nih-at/libzip/releases/download/v1.7.3/libzip-1.7.3.tar.gz" \
    && tar -xzvf libzip-1.7.3.tar.gz \
    && cd libzip-1.7.3 \
    && mkdir out \
    && cd out \
    && cmake -DCMAKE_INSTALL_PREFIX:PATH=${LOCAL_D}/libzip ../ \
    && make \
    && make install \
    && echo "${LOCAL_D}/libzip/lib64" > ${LD_CONF_D}/libzip.conf \
    && ldconfig \
    && rm -rf ${SRC_DIR}/libzip*

#Install onigiguruma for php 8.0.3
RUN cd ${SRC_DIR} \
    && wget "https://github.com/kkos/oniguruma/releases/download/v6.9.6/onig-6.9.6.tar.gz" \
    && tar -xzvf onig-6.9.6.tar.gz \
    && cd onig-6.9.6 \
    && ./configure --prefix="${LOCAL_D}/oniguruma" \
    && make \
    && make install \
    && echo "${LOCAL_D}/oniguruma/lib" > ${LD_CONF_D}/oniguruma.conf \
    && ldconfig \
    && rm -rf ${SRC_DIR}/onig*

#Install php
ENV PHP_HOME=${LOCAL_D}/php
ENV PHP_INI_DIR=${PHP_HOME}/lib
ENV PHP_CONF_D=${PHP_HOME}/lib/php.conf.d
ENV PHP_VERSION=8.0.3

RUN cd ${SRC_DIR} \
    && wget "https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz" \
    && tar -xzvf php-${PHP_VERSION}.tar.gz \
    && cd php-${PHP_VERSION} \
    &&  ./configure --prefix="${LOCAL_D}/php" --enable-calendar --with-bz2 --with-zlib --with-curl \
        --with-zip --enable-soap --enable-sockets --with-expat --without-sqlite3 --without-pdo-sqlite \
        --enable-mbstring --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-soap --enable-simplexml --enable-session \
        --enable-xml --enable-opcache --enable-xmlreader --enable-xmlwriter --enable-zts --enable-mysqlnd \
        --with-mysqli=mysqlnd --with-pdo-mysql --with-pdo-mysql=mysqlnd --with-pdo-oci="instantclient,${ORACLE_INSTANTCLIENT_HOME}" \
        --with-oci8="instantclient,${ORACLE_INSTANTCLIENT_HOME}" --enable-intl --with-apxs2="${LOCAL_D}/apache2/bin/apxs" --enable-exif \
        --with-openssl --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data --enable-gd --enable-ctype \
        --with-jpeg --with-webp --with-freetype --enable-gd-jis-conv --with-gettext --enable-libgcc --enable-bcmath \
        --enable-intl --with-config-file-scan-dir="${PHP_CONF_D}" \
        "PKG_CONFIG_PATH=${PKG_CONFIG_PATH}:${LOCAL_D}/libzip/lib64/pkgconfig/:${LOCAL_D}/oniguruma/lib/pkgconfig/" \
    && make -j ${N_PROC} \
    && make install \
    && cp php.ini-production ${PHP_INI_DIR}/php.ini \
    && groupadd www-data \
    && useradd -r -g www-data -s /bin/false www-data \
    && ln -s ${PHP_HOME}/bin/php ${LOCAL_D}/bin/ \
    && ln -s ${PHP_HOME}/bin/phpize ${LOCAL_D}/bin/ \
    && ln -s ${PHP_HOME}/bin/php-config ${LOCAL_D}/bin/ \
    && rm -rf ${SRC_DIR}/php* \
    && mkdir ${PHP_CONF_D} \
# smoke test
    # Opcache, jit config
    && echo "zend_extension=opcache;" > ${PHP_CONF_D}/opcache.ini \
    && echo "opcache.enable=1" >> ${PHP_CONF_D}/opcache.ini \
    && echo "opcache.jit_buffer_size=100M" >> ${PHP_CONF_D}/opcache.ini \
    && echo "opcache.jit=1255" >> ${PHP_CONF_D}/opcache.ini \
    # End, Opcache, jit config
    && echo "short_open_tag=On" > ${PHP_CONF_D}/php.custom.ini \
    && echo "display_errors=On" >> ${PHP_CONF_D}/php.custom.ini \
    && echo "display_startup_errors=On" >> ${PHP_CONF_D}/php.custom.ini \
	&& php -v

#mongo client
RUN cd ${SRC_DIR} \
    && git clone "https://github.com/mongodb/mongo-php-driver.git" \
    && cd mongo-php-driver \
    && git checkout 1.9.0 \
    && git submodule update --init \
    && phpize  \
    && ./configure \
    && make -j ${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/mongo-php-driver \
    && echo "extension=mongodb.so" > ${PHP_CONF_D}/mongo.ini

#redis client
RUN cd ${SRC_DIR} \
    && git clone "https://github.com/phpredis/phpredis.git" \
    && cd phpredis \
    && git checkout 5.3.4 \
    && phpize \
    && ./configure  \
    && make -j ${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/phpredis \
    && echo "extension=redis.so" > ${PHP_CONF_D}/redis.ini

#Install parellel (new pthread)
RUN cd ${SRC_DIR} \
    && git clone "https://github.com/krakjoe/parallel.git" \
    && cd parallel \
    && phpize \
    && ./configure --enable-parallel \
    && make -j ${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/parallel \
    && echo "extension=parallel.so" > ${PHP_CONF_D}/parallel.ini

#Install librdkafka client for kafka client
RUN cd ${SRC_DIR} \
    && git clone "https://github.com/edenhill/librdkafka" \
    && cd librdkafka \
    && git checkout v1.6.1 \
    && ./configure --prefix="${LOCAL_D}/librdkafka" \
    && make -j ${N_PROC} \
    && make install \
    && echo "${LOCAL_D}/librdkafka/lib" > ${LD_CONF_D}/librdkafka.conf \
    && ldconfig \
    && rm -rf ${SRC_DIR}/librdkafka*

# kafka client
RUN cd ${SRC_DIR} \
    && wget "https://github.com/arnaud-lb/php-rdkafka/archive/refs/tags/5.0.0.tar.gz" \
    && tar -xzvf 5.0.0.tar.gz  \
    && cd php-rdkafka-5.0.0 \
    && phpize \
    && ./configure --with-rdkafka=${LOCAL_D}/librdkafka \
    && make all -j ${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/php-rdkafka* ${SRC_DIR}/*.tar.gz \
    && echo "extension=rdkafka.so" > ${PHP_CONF_D}/rdkafka.ini

#Install composer
RUN cd ${SRC_DIR} \
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && php -r "unlink('composer-setup.php');" \
    && rm -f ${SRC_DIR}/composer.phar

COPY httpd-conf/httpd.conf ${APACHE_HOME}/conf/
COPY httpd-foreground /usr/local/bin/
CMD ["httpd-foreground"]