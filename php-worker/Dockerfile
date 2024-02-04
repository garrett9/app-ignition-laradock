#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

ARG LARADOCK_PHP_VERSION
FROM php:${LARADOCK_PHP_VERSION}-alpine

LABEL maintainer="Mahmoud Zalt <mahmoud@zalt.me>"

ARG LARADOCK_PHP_VERSION

# If you're in China, or you need to change sources, will be set CHANGE_SOURCE to true in .env.

ARG CHANGE_SOURCE=false
RUN if [ ${CHANGE_SOURCE} = true ]; then \
  # Change application source from dl-cdn.alpinelinux.org to aliyun source
  sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories \
  ;fi

RUN apk --update add wget \
  curl \
  git \
  build-base \
  libmcrypt-dev \
  libxml2-dev \
  linux-headers \
  pcre-dev \
  zlib-dev \
  autoconf \
  cyrus-sasl-dev \
  libgsasl-dev \
  oniguruma-dev \
  supervisor; \
  if [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "50600" ] || \
     [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "70000" ]; then \
    apk --update add libressl libressl-dev; \
  else \
    apk --update add openssl-dev; \
  fi


RUN pecl channel-update pecl.php.net; \
  docker-php-ext-install mysqli mbstring pdo pdo_mysql xml pcntl; \
  if [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "80100" ] || \
     [ $(php -r "echo PHP_VERSION_ID - PHP_RELEASE_VERSION;") = "80200" ]; then \
    php -m | grep -oiE '^tokenizer$'; \
  else \
    docker-php-ext-install tokenizer; \
  fi

# Add a non-root user:
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN addgroup -g ${PGID} laradock && \
  adduser -D -G laradock -u ${PUID} laradock

#Install BZ2:
ARG INSTALL_BZ2=false
RUN if [ ${INSTALL_BZ2} = true ]; then \
  apk --update add bzip2-dev; \
  docker-php-ext-install bz2; \
  fi

###########################################################################
# PHP GnuPG:
###########################################################################

ARG INSTALL_GNUPG=false

RUN set -eux; \
  if [ ${INSTALL_GNUPG} = true ]; then \
    apk add --no-cache --no-progress --virtual BUILD_DEPS_PHP_GNUPG gpgme-dev; \
    if [ ${LARADOCK_PHP_VERSION} = "7.0" ]; then \
      wget -O /tmp/gnupg.tgz "https://pecl.php.net/get/gnupg-1.5.1.tgz"; \
      mkdir -p /tmp/gnupg; \
      tar -C /tmp/gnupg -zxvf /tmp/gnupg.tgz --strip 1; \
      cd /tmp/gnupg; \
      phpize; \
      ./configure; \
      make && make install; \
    else \
      pecl install gnupg; \
    fi; \
    docker-php-ext-enable gnupg; \
    php -m | grep -oiE '^gnupg$'; \
  fi

#Install LDAP
ARG INSTALL_LDAP=false;
RUN set -eux; if [ ${INSTALL_LDAP} = true ]; then \
  apk add --no-cache --no-progress openldap-dev; \
  docker-php-ext-install ldap; \
  php -m | grep -oiE '^ldap$'; \
  fi

#Install ImageMagick:
ARG INSTALL_IMAGEMAGICK=false
ARG IMAGEMAGICK_VERSION=latest
ENV IMAGEMAGICK_VERSION ${IMAGEMAGICK_VERSION}
RUN set -eux; \
  if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
  apk add --update --no-cache imagemagick-dev imagemagick; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  cd /tmp && \
  if [ ${IMAGEMAGICK_VERSION} = "latest" ]; then \
  git clone https://github.com/Imagick/imagick; \
  else \
  git clone --branch ${IMAGEMAGICK_VERSION} https://github.com/Imagick/imagick; \
  fi && \
  cd imagick && \
  phpize && \
  ./configure && \
  make && \
  make install && \
  rm -r /tmp/imagick; \
  else \
  pecl install imagick; \
  fi && \
  docker-php-ext-enable imagick; \
  php -m | grep -q 'imagick'; \
  fi

#Install GMP package:
RUN apk add --update --no-cache gmp gmp-dev \
  && docker-php-ext-install gmp

#Install BCMath package:
ARG INSTALL_BCMATH=false
RUN if [ ${INSTALL_BCMATH} = true ]; then \
  docker-php-ext-install bcmath \
  ;fi

#Install SOAP package:
ARG INSTALL_SOAP=false
RUN if [ ${INSTALL_SOAP} = true ]; then \
  docker-php-ext-install soap \
  ;fi

# Install PostgreSQL drivers:
ARG INSTALL_PGSQL=false
RUN if [ ${INSTALL_PGSQL} = true ]; then \
  apk --update add postgresql-dev \
  && docker-php-ext-install pdo_pgsql \
  ;fi

# Install ZipArchive:
ARG INSTALL_ZIP_ARCHIVE=false
RUN set -eux; \
  if [ ${INSTALL_ZIP_ARCHIVE} = true ]; then \
  apk --update add libzip-dev && \
  if [ ${LARADOCK_PHP_VERSION} = "7.3" ] || [ ${LARADOCK_PHP_VERSION} = "7.4" ] || [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  docker-php-ext-configure zip; \
  else \
  docker-php-ext-configure zip --with-libzip; \
  fi && \
  # Install the zip extension
  docker-php-ext-install zip \
  ;fi

# Install MySQL Client:
ARG INSTALL_MYSQL_CLIENT=false
RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
  apk --update add mysql-client \
  ;fi

# Install FFMPEG:
ARG INSTALL_FFMPEG=false
RUN if [ ${INSTALL_FFMPEG} = true ]; then \
  apk --update add ffmpeg \
  ;fi

# Install BBC Audio Waveform Image Generator:
ARG INSTALL_AUDIOWAVEFORM=false
RUN if [ ${INSTALL_AUDIOWAVEFORM} = true ]; then \
  apk add git make cmake gcc g++ libmad-dev libid3tag-dev libsndfile-dev gd-dev boost-dev libgd libpng-dev zlib-dev \
  && apk add autoconf automake libtool gettext \
  && wget https://github.com/xiph/flac/archive/1.3.3.tar.gz \
  && tar xzf 1.3.3.tar.gz \
  && cd flac-1.3.3 \
  && ./autogen.sh \
  && ./configure --enable-shared=no \
  && make \
  && make install \
  && cd .. \
  && git clone https://github.com/bbc/audiowaveform.git \
  && cd audiowaveform \
  && wget https://github.com/google/googletest/archive/release-1.10.0.tar.gz \
  && tar xzf release-1.10.0.tar.gz \
  && ln -s googletest-release-1.10.0/googletest googletest \
  && ln -s googletest-release-1.10.0/googlemock googlemock \
  && mkdir build \
  && cd build \
  && cmake .. \
  && make \
  && make install \
  ;fi

# Install APCU ext
ARG INSTALL_APCU=false

RUN if [ ${INSTALL_APCU} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        pecl install -a apcu-4.0.11; \
    else \
        pecl install apcu; \
    fi && \
    docker-php-ext-enable apcu \
;fi

ARG INSTALL_GHOSTSCRIPT=false
RUN if [ $INSTALL_GHOSTSCRIPT = true ]; then \
  apk --update add ghostscript \
  ;fi

# Install Redis package:
ARG INSTALL_REDIS=false
RUN if [ ${INSTALL_REDIS} = true ]; then \
  # Install Redis Extension
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
  printf "\n" | pecl install -o -f redis-4.3.0; \
  else \
  printf "\n" | pecl install -o -f redis; \
  fi; \
  rm -rf /tmp/pear; \
  docker-php-ext-enable redis \
  ;fi

###########################################################################
# xlswriter:
###########################################################################

ARG INSTALL_XLSWRITER=false

RUN set -eux; \
    if [ ${INSTALL_XLSWRITER} = true ]; then \
      # Install Php xlswriter Extension \
      if   [ $(php -r "echo PHP_MAJOR_VERSION;") != "5" ]; then \
          pecl install xlswriter && \
          docker-php-ext-enable xlswriter && \
          php -m | grep -q 'xlswriter'; \
      else \
          echo "PHP Extension for xlswriter is not supported for PHP 5.0"; \
      fi \
    ;fi

###########################################################################
# Imap EXTENSION
###########################################################################

ARG INSTALL_IMAP=false

RUN if [ ${INSTALL_IMAP} = true ]; then \
  apk add --update imap-dev && \
  docker-php-ext-configure imap --with-imap --with-imap-ssl && \
  docker-php-ext-install imap \
  ;fi

###########################################################################
# XMLRPC:
###########################################################################

ARG INSTALL_XMLRPC=false

RUN if [ ${INSTALL_XMLRPC} = true ]; then \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  pecl install xmlrpc-1.0.0RC3; \
  docker-php-ext-enable xmlrpc; \
  else \
  docker-php-ext-install xmlrpc; \
  fi; \
  php -m | grep -r 'xmlrpc'; \
  fi

###########################################################################
# PHP Memcached:
###########################################################################

ARG INSTALL_MEMCACHED=false

RUN if [ ${INSTALL_MEMCACHED} = true ]; then \
  apk --update add libmemcached-dev; \
  # Install the php memcached extension
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
  pecl install memcached-2.2.0; \
  else \
  pecl install memcached; \
  fi; \
  docker-php-ext-enable memcached; \
  php -m | grep -r 'memcached'; \
  fi

###########################################################################
# Intl:
###########################################################################

ARG INSTALL_INTL=false

RUN if [ ${INSTALL_INTL} = true ]; then \
    apk add icu-dev && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl && \
    docker-php-ext-enable intl \
;fi

#
#--------------------------------------------------------------------------
# Optional Supervisord Configuration
#--------------------------------------------------------------------------
#
# Modify the ./supervisor.conf file to match your App's requirements.
# Make sure you rebuild your container with every change.
#

COPY supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# If you need to modify this image, feel free to do it right here.
#
# -- Your awesome modifications go here -- #

#
#--------------------------------------------------------------------------
# Check PHP version
#--------------------------------------------------------------------------
#

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

# Clean up
RUN rm /var/cache/apk/* \
  && mkdir -p /var/www

WORKDIR /etc/supervisor/conf.d/
