FROM php:8.1-fpm-alpine3.14


RUN deluser www-data && adduser -u 1005 --disabled-password www-data 

# RUN apk add --update --no-cache zlib-dev  libpng-dev freetype-dev libjpeg-turbo-dev curl-dev aws-cli\
#     icu-dev oniguruma-dev libxslt-dev libzip-dev \
#     && docker-php-ext-configure gd \
#         --with-freetype-dir=/usr/lib/ \
#         --with-png-dir=/usr/lib/ \
#         --with-jpeg-dir=/usr/lib/ \
#         --with-gd \
#     && docker-php-ext-install -j$(nproc) gd \
#     && docker-php-ext-install -j$(nproc) bcmath intl  pdo_mysql soap zip xsl sockets

RUN curl -sSLf \
        -o /usr/local/bin/install-php-extensions \
        https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions && \
    chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions gd 

RUN install-php-extensions bcmath intl pdo_mysql soap sockets  xsl zip
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer 
#COPY ./magento2 /var/www/html
COPY ./files/php.ini "$PHP_INI_DIR/php.ini"
COPY magento2 /var/www/html
