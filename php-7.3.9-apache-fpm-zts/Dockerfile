#
# docker build -t ninjabenz/php:7.3.9-apache-fpm-zts .
# docker build --no-cache -t ninjabenz/php:7.3.9-apache-fpm-zts .
#
FROM centos:7.6.1810

ENV PHPIZE_DEPS="libmcrypt-devel libicu-devel bzip2-devel curl-devel libxml2-devel bison-devel libzip-devel libaio-devel openssl-devel zlib-devel libpng-devel libjpeg-devel"
ENV N_PROC=4

RUN yum groupinstall -y "Development tools" \
    && yum install -y wget expat-devel gcc gcc-c++ glibc cmake ${PHPIZE_DEPS} \
    && yum clean all
        
ENV SRC_DIR=/usr/local/src 
ENV LOCAL_D=/usr/local
ENV LD_CONF_D=/etc/ld.so.conf.d

#Install cmake
RUN cd ${SRC_DIR} \
    && wget "https://github.com/Kitware/CMake/releases/download/v3.15.3/cmake-3.15.3.tar.gz" \
    && tar -xzvf cmake-3.15.3.tar.gz \
    && cd cmake-3.15.3 \
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

#Install nghttp2 for apache
RUN cd ${SRC_DIR} \
    && wget "https://github.com/nghttp2/nghttp2/releases/download/v1.39.2/nghttp2-1.39.2.tar.gz" \
    && tar -xzvf nghttp2-1.39.2.tar.gz \
    && cd nghttp2-1.39.2 \
    && ./configure --prefix="${LOCAL_D}/nghttp2" \
    && make -j${N_PROC} \
    && make install \
    && rm -rf ${SRC_DIR}/nghttp2-1.39.2* \
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
    && rm -rf ${SRC_DIR}/brotli \
    && echo "${LOCAL_D}/brotli/lib" > ${LD_CONF_D}/brotli.conf \
    && ldconfig

#Install apache
ENV APACHE_HOME=${LOCAL_D}/apache2
RUN cd ${SRC_DIR} \
    && wget "http://www.apache.org/dist/httpd/httpd-2.4.41.tar.gz" \
    && tar -xzvf httpd-2.4.41.tar.gz \
    && cd httpd-2.4.41 \
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
    && ln -sf /dev/stdout "${APACHE_HOME}/access_log" \
    && ln -sf /dev/stderr "${APACHE_HOME}/error_log" \
    && ln -s "${APACHE_HOME}/bin/apachectl" /usr/local/bin/ \
    && ln -s "${APACHE_HOME}/bin/httpd" /usr/local/bin/ \
# smoke test
	&& apachectl -v

# https://httpd.apache.org/docs/2.4/stopping.html#gracefulstop
STOPSIGNAL WINCH

EXPOSE 80 443 9000
WORKDIR ${APACHE_HOME}/htdocs

#Install oracle instantclient
ENV ORACLE_INSTANTCLIENT_HOME=${LOCAL_D}/instantclient_19_3
RUN cd ${LOCAL_D} \
    && wget "https://raw.githubusercontent.com/ekanant/oracle_instantclient/master/19.3.0.0.0/instantclient-basic-linux.x64-19.3.0.0.0dbru.zip" \
    && wget "https://raw.githubusercontent.com/ekanant/oracle_instantclient/master/19.3.0.0.0/instantclient-sdk-linux.x64-19.3.0.0.0dbru.zip" \
    && unzip instantclient-basic-*.zip \
    && unzip instantclient-sdk-*.zip \
    && rm -rf instantclient-*.zip \ 
    && echo "${ORACLE_INSTANTCLIENT_HOME}" > ${LD_CONF_D}/oracle_instantclient.conf \
    && ldconfig

#Install lib icu
RUN cd ${SRC_DIR} \
    && wget "https://github.com/unicode-org/icu/releases/download/release-64-2/icu4c-64_2-src.tgz" \
    && tar -xzvf icu4c-64_2-src.tgz \
    && cd icu/source \
    && ./configure --prefix="${LOCAL_D}/icu" \
    && make -j${N_PROC} \
    && make install \
    && echo "${LOCAL_D}/icu/lib" > ${LD_CONF_D}/icu.conf \
    && ldconfig \
    && cd ${SRC_DIR} \
    && rm -rf icu*

RUN cd ${SRC_DIR} \
    && wget "https://github.com/nih-at/libzip/archive/rel-1-4-0.tar.gz" \
    && tar -xzvf rel-1-4-0.tar.gz \
    && cd libzip-rel-1-4-0 \
    && mkdir out \
    && cd out \
    && cmake -DCMAKE_INSTALL_PREFIX:PATH=${LOCAL_D}/libzip ../ \
    && make -j${N_PROC} \
    && make install \
    && echo "${LOCAL_D}/libzip/lib" > ${LD_CONF_D}/libzip.conf \
    && cd ${SRC_DIR} \
    && rm -rf *.tar.gz \
    && rm -rf libzip*

#Install php
ENV PHP_HOME=${LOCAL_D}/php
ENV PHP_INI_DIR=${PHP_HOME}/lib
ENV PHP_VERSION=7.3.9

RUN cd ${SRC_DIR} \
    && wget "https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz" \
    && tar -xzvf php-${PHP_VERSION}.tar.gz \
    && cd php-${PHP_VERSION} \
    &&  ./configure --prefix="${LOCAL_D}/php" --enable-calendar --with-bz2 --with-zlib --with-curl \
        --with-libzip="${LOCAL_D}/libzip" --enable-zip \
        --enable-mbstring --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-soap --enable-simplexml --enable-session \
        --enable-xml --enable-wddx --enable-opcache --enable-json --enable-xmlreader --enable-xmlwriter --enable-maintainer-zts --enable-mysqlnd \
        --with-mysqli=mysqlnd --with-pdo-mysql --with-pdo-mysql=mysqlnd --with-pdo-oci="instantclient,${ORACLE_INSTANTCLIENT_HOME}" \
        --with-oci8="instantclient,${ORACLE_INSTANTCLIENT_HOME}" --enable-intl --with-apxs2="${LOCAL_D}/apache2/bin/apxs" --enable-exif \
        --with-tsrm-pthreads --with-openssl --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data --with-gd --enable-ctype \
        --with-jpeg-dir --with-png-dir --enable-gd-jis-conv --with-gettext --with-xmlrpc --enable-libgcc --with-libxml-dir --enable-bcmath \
        --with-icu-dir="${LOCAL_D}/icu" \
    && make -j${N_PROC} \
    && make install \
    && cp php.ini-production ${PHP_INI_DIR}/php.ini \
    && groupadd www-data \
    && useradd -r -g www-data -s /bin/false www-data \
    && ln -s ${PHP_HOME}/bin/php ${LOCAL_D}/bin/ \
    && ln -s ${PHP_HOME}/bin/phpize ${LOCAL_D}/bin/ \
    && rm -rf ${SRC_DIR}/php* \
# smoke test
	&& php -v

#Install pthreads
RUN cd ${LOCAL_D} \
    && git clone https://github.com/krakjoe/pthreads.git \
    && cd pthreads \
	&& phpize \
	&& ./configure --prefix="${LOCAL_D}/pthreads" --with-php-config="${PHP_HOME}/bin/php-config" \
    && make -j${N_PROC} \
    && make install

COPY httpd-conf/httpd.conf ${APACHE_HOME}/conf/
COPY php-conf/php*.ini ${LOCAL_D}/php/lib/
COPY httpd-foreground /usr/local/bin/
CMD ["httpd-foreground"]