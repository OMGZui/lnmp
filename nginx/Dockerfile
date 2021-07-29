FROM nginx:alpine

COPY nginx.conf /etc/nginx

USER root
RUN mkdir -p /var/www/demo && echo "<?php phpinfo();" > /var/www/demo/index.php
RUN adduser -D -u 1000 www-data -G www-data

EXPOSE 80