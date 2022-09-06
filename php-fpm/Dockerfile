FROM php:8.1-fpm-alpine3.14


RUN deluser www-data && adduser -u 1005 --disabled-password www-data 
RUN curl -sSLf \
        -o /usr/local/bin/install-php-extensions \
        https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions && \
    chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions gd 

RUN install-php-extensions bcmath intl pdo_mysql soap sockets  xsl zip
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer 
#COPY php-fpm/magento2 /var/www/html
COPY php-fpm/files/php.ini "$PHP_INI_DIR/php.ini"
