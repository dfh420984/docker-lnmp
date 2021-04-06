FROM php:7.4-fpm

# ARGS
ARG CHANGE_SOURCE=false
ARG XDEBUG_REMOTE_HOST=127.0.0.1
ARG XDEBUG_REMOTE_PORT=9000

# Change Timezone
ARG TIME_ZONE=UTC
ENV TIME_ZONE ${TIME_ZONE}
RUN ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone

# Change China Sources
COPY zhongkeda.list /etc/apt/china.sources.list
RUN if [ ${CHANGE_SOURCE} = true ]; then \
	mv /etc/apt/sources.list /etc/apt/source.list.bak && mv /etc/apt/china.sources.list /etc/apt/sources.list \
;fi

RUN apt-get -o Acquire::Check-Valid-Until=false update && apt-get -y upgrade

# Install Base Components
RUN apt-get install -y --no-install-recommends default-mysql-client libpq-dev libfreetype6-dev libjpeg62-turbo-dev libpng-dev  ntpdate cron vim unzip git procps net-tools telnet

####################################################################################
# Install Extension
####################################################################################

RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install gd pdo_mysql mysqli pgsql pdo_pgsql pcntl

# Redis
COPY redis-4.1.1.tgz /home/redis.tgz
RUN pecl install /home/redis.tgz \
	&& echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini

# Mcrypt
COPY mcrypt-1.0.4.tgz /home/mcrypt.tgz
RUN apt-get install -y libmcrypt-dev libmhash-dev \
	&& pecl install /home/mcrypt.tgz \
	&& echo "extension=mcrypt.so" > /usr/local/etc/php/conf.d/mcrypt.ini

# PHP Composer
COPY composer.phar /usr/local/bin/composer
RUN chmod 755 /usr/local/bin/composer

# Swoole
COPY swoole-4.6.4.tgz /home/swoole.tgz
RUN pecl install /home/swoole.tgz \
	&& echo "extension=swoole.so" > /usr/local/etc/php/conf.d/swoole.ini

#xdebug
COPY xdebug-3.0.3.tgz /home/xdebug.tgz
RUN pecl install /home/xdebug.tgz \
	&& echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
	&& echo "xdebug.idekey=docker" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_host=${XDEBUG_REMOTE_HOST}" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_port=${XDEBUG_REMOTE_PORT}" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=on" >> /usr/local/etc/php/conf.d/xdebug.ini

# Setup Composer Source
RUN if [ $CHANGE_SOURCE = true ]; then \
	composer config  -g repo.packagist composer https://packagist.laravel-china.org \
;fi


ENV LC_ALL C.UTF-8

ENTRYPOINT ["/bin/bash", "/home/start.sh"]
