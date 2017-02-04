#build
#docker build -t ninjabenz/php:5.6.23-apache --no-cache --pull .
#

FROM ninjabenz/php:5.6.23-apache-zts-base

MAINTAINER Ekanant<ekanant2535@gmail.com>

# Get repository and install wget and vim
RUN apt-get update && apt-get install --no-install-recommends -y \
        wget \
        vim \
        nano \
        git \
        unzip

# Add PostgreSQL repository
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ jessie-pgdg main" > /etc/apt/sources.list.d/pgdg.list
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
      apt-key add -

# Install Oracle Instantclient
COPY instantclient-*.zip /opt/oracle/

RUN cd /opt/oracle \
    && unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
    && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
    && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
    && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
    && rm -rf /opt/oracle/*.zip

#RUN mkdir /opt/oracle \
#    && cd /opt/oracle \
#    && wget https://s3.amazonaws.com/merofile/instantclient-basic-linux.x64-12.1.0.2.0.zip \
#    && wget https://s3.amazonaws.com/merofile/instantclient-sdk-linux.x64-12.1.0.2.0.zip \
#    && unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
#    && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
#    && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
#    && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
#    && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
#    && rm -rf /opt/oracle/*.zip


# Install PHP extensions deps
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        postgresql-server-dev-9.5 \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        zlib1g-dev \
        libicu-dev \
        unixodbc-dev \
        libxml2-dev \
        libaio-dev \
        libmemcached-dev \
        freetds-dev

#Install facebook zstd compress
RUN git clone --recursive --depth=1 https://github.com/kjdev/php-ext-zstd.git \
    && cd php-ext-zstd \
    && phpize \
    && ./configure --enable-zstd \
    && make -j$(nproc) \
    && make install \
    && docker-php-ext-enable zstd \
    && cd .. \
    && rm -rf php-ext-zstd/

# Install PHP extensions
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.0.10 \
    && pecl install apcu-4.0.10 \
    && pecl install redis-2.2.7 \
    && pecl install memcached \
    && pecl install mongo \
		&& pecl install pthreads-2.0.10 \
    && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_1,12.1 \
    && docker-php-ext-configure pdo_dblib --with-libdir=/lib/x86_64-linux-gnu \
    && docker-php-ext-install \
            iconv \
            mbstring \
            intl \
            mcrypt \
            gd \
            pgsql \
            mysql \
            mysqli \
            pdo_pgsql \
            pdo_mysql \
            pdo_oci \
            pdo_dblib \
            soap \
            sockets \
            zip \
            pcntl \
    && docker-php-ext-enable \
            oci8 \
            apcu \
            memcached \
            redis \
            opcache \
            mongo \
						pthreads

# Clean repository
RUN apt-get clean \
    && apt-get autoclean \
    && rm -rf /usr/share/doc \
    && rm -rf /var/lib/apt/lists/*

# Mapping apache htdocs path
VOLUME /var/www/html

#Mapping apache log path
VOLUME /var/log/apache2
