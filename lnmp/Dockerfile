FROM php:7.2-fpm

WORKDIR /data/website

RUN curl -L -o /tmp/redis-5.3.3.tgz  https://pecl.php.net/get/redis-5.3.3.tgz && \
    docker-php-source extract &&\
    cd /tmp &&\
    tar -zxvf /tmp/redis-5.3.3.tgz && \
    mv /tmp/redis-5.3.3 /usr/src/php/ext/phpredis && \
    docker-php-ext-install phpredis && \
    pecl install -o -f swoole && \
    docker-php-ext-enable swoole


