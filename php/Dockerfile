FROM php:7.4-fpm

# 添加源
RUN sed -i 's/deb.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.aliyun.com/' /etc/apt/sources.list
RUN apt-get update && \
        pecl channel-update pecl.php.net

# phpredis
COPY redis-4.2.0.tgz /home
RUN pecl install /home/redis-4.2.0.tgz && \
    docker-php-ext-enable redis && \
    rm /home/redis-4.2.0.tgz

# mongodb
COPY mongodb-1.5.3.tgz /home
RUN pecl install /home/mongodb-1.5.3.tgz && \
    docker-php-ext-enable mongodb && \
    rm /home/mongodb-1.5.3.tgz

# opcache
RUN docker-php-ext-install opcache
COPY opcache.ini /usr/local/etc/php/conf.d/opcache.ini

# php配置文件
ADD php.ini    /usr/local/etc/php/php.ini

USER root

# 清理
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

RUN usermod -u 1000 www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000