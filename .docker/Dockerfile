FROM ubuntu:xenial

LABEL maintainer="Deepak Gupta <dieepak@gmail.com>"

# Fix debconf warnings upon build
ENV DEBIAN_FRONTEND=noninteractive

ARG TIMEZONE

# Manually set up the apache environment variables
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APPLICATION_DOCUMENT_ROOT /var/www/html/public

ENV OS_LOCALE="en_US.UTF-8"
RUN apt-get update && apt-get install -y locales && locale-gen ${OS_LOCALE}
ENV LANG=${OS_LOCALE} \
    LANGUAGE=${OS_LOCALE} \
    LC_ALL=${OS_LOCALE} \
    DEBIAN_FRONTEND=noninteractive

ENV APACHE_CONF_DIR=/etc/apache2 \
    PHP_CONF_DIR=/etc/php/7.2 \
    PHP_DATA_DIR=/var/lib/php

COPY entrypoint.sh /sbin/entrypoint.sh

RUN	\
	BUILD_DEPS='software-properties-common python-software-properties' \
    && dpkg-reconfigure locales \
	&& apt-get install --no-install-recommends -y $BUILD_DEPS \
	&& add-apt-repository -y ppa:ondrej/php \
	&& add-apt-repository -y ppa:ondrej/apache2 \
	&& apt-get update \
    && apt-get install -y mysql-client vim curl apache2 libapache2-mod-php7.2 php-memcached php7.2-mysql php7.2-pgsql php-redis php7.2-sqlite3 php-xdebug php7.2-bcmath php7.2-bz2 php7.2-dba php7.2-enchant php7.2-gd php7.2-gmp php-igbinary php-imagick php7.2-imap php7.2-interbase php7.2-intl php7.2-ldap php-mongodb php-msgpack php7.2-odbc php7.2-phpdbg php7.2-pspell php-raphf php7.2-recode php7.2-snmp php7.2-soap php-ssh2 php7.2-sybase php-tideways php7.2-tidy php7.2-xmlrpc php7.2-xsl php-yaml php-zmq \
    # Apache settings
    && cp /dev/null ${APACHE_CONF_DIR}/conf-available/other-vhosts-access-log.conf \
    && rm ${APACHE_CONF_DIR}/sites-enabled/000-default.conf ${APACHE_CONF_DIR}/sites-available/000-default.conf \
    && a2enmod rewrite php7.2 \
	# Install composer
	&& curl -sS https://getcomposer.org/installer | php -- --version=1.6.4 --install-dir=/usr/local/bin --filename=composer \
	# Cleaning
	&& apt-get purge -y --auto-remove $BUILD_DEPS \
	&& apt-get autoremove -y \
	&& rm -rf /var/lib/apt/lists/* \
	# Forward request and error logs to docker log collector
	&& ln -sf /dev/stdout /var/log/apache2/access.log \
	&& ln -sf /dev/stderr /var/log/apache2/error.log \
	&& chmod 755 /sbin/entrypoint.sh \
	&& chown www-data:www-data ${PHP_DATA_DIR} -Rf

COPY ./config/apache2.conf ${APACHE_CONF_DIR}/apache2.conf
COPY ./config/apache-config.conf ${APACHE_CONF_DIR}/sites-enabled/apache-config.conf
COPY ./config/php-ini-overrides.ini  ${PHP_CONF_DIR}/apache2/conf.d/php-ini-overrides.ini

WORKDIR /var/www/html

# By default, simply start apache.
CMD ["/sbin/entrypoint.sh"]