FROM ubuntu/apache2:2.4-22.04_beta

WORKDIR /var/www/

RUN apt update && \
  apt install openssl

COPY vhost.conf /etc/apache2/sites-enabled/vhost.conf

RUN a2enmod rewrite proxy_html proxy_http xml2enc ssl http2 proxy_fcgi

EXPOSE 80 443
