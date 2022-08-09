FROM amatkovskiy/sigma_magento-nginx:1.23.0-alpine
#FROM nginx:1.21.6-alpine
RUN adduser -D -H -u 1005 -s /bin/bash -G www-data www-data
COPY nginx/config/templates/ /etc/nginx/templates/
RUN mkdir -p /custom_checks/fack_healthcheck
COPY index.html /custom_checks/fack_healthcheck/
RUN mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.off
