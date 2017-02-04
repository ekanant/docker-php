#build
#docker build -t ninjabenz/php:5.6-fpm-zts-alpine --no-cache --pull .
#

FROM ninjabenz/php:5.6-fpm-zts-alpine-base

MAINTAINER Ekanant<ekanant2535@gmail.com>

RUN apk --no-cache add \
      wget \
      unzip \
      curl \
      g++ \
      gcc \
      musl \
# PHP Extension dependencies
      re2c \
      autoconf \
      make \
      file

# Install Oracle Instantclient
ENV ORACLE_HOME=/opt/oracle/instantclient_12_1

COPY instantclient-*.zip /opt/oracle/
RUN unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && ln -s $ORACLE_HOME/libclntsh.so.12.1 $ORACLE_HOME/libclntsh.so \
    && ln -s $ORACLE_HOME/libclntshcore.so.12.1 $ORACLE_HOME/libclntshcore.so \
    && ln -s $ORACLE_HOME/libocci.so.12.1 $ORACLE_HOME/libocci.so \
    && rm -rf /opt/oracle/*.zip

#Install facebook zstd compress
COPY facebook-zstd.zip /opt/facebook-zstd/
RUN unzip /opt/facebook-zstd/facebook-zstd.zip -d /opt/facebook-zstd/ \
    && cd /opt/facebook-zstd/php-ext-zstd/ \
    && phpize \
    && ./configure --enable-zstd \
    && make -j$(nproc) \
    && make install \
    && rm -rf /opt/facebook-zstd/


# Install PHP extensions
RUN echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.0.12 \
    && docker-php-ext-enable oci8

# Clean repository
RUN rm -rf /var/cache/apk/* \
    && rm -rf /usr/share/doc
