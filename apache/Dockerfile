FROM php:5.6-apache
MAINTAINER Michael Babker <michael.babker@mautic.org> (@mbabker)

# Install PHP extensions
RUN apt-get update && apt-get install --no-install-recommends -y \
    cron \
    libc-client-dev \
    libicu-dev \
    libkrb5-dev \
    libmcrypt-dev \
    libssl-dev \
    unzip \
    zip \
    && rm /etc/cron.daily/*
RUN docker-php-ext-configure imap --with-imap --with-imap-ssl --with-kerberos \
    && docker-php-ext-install imap intl mbstring mcrypt mysqli pdo pdo_mysql zip

VOLUME /var/www/html

# Define Mautic version and expected SHA1 signature
ENV MAUTIC_VERSION 2.7.1
ENV MAUTIC_SHA1 09e39b96139e15d9fbe4e96d88ff5532ace92206
ENV MAUTIC_RUN_CRON_JOBS true

# Download package and extract to web volume
RUN curl -o mautic.zip -SL https://s3.amazonaws.com/mautic/releases/${MAUTIC_VERSION}.zip \
	&& echo "$MAUTIC_SHA1 *mautic.zip" | sha1sum -c - \
	&& mkdir /usr/src/mautic \
	&& unzip mautic.zip -d /usr/src/mautic \
	&& rm mautic.zip \
	&& chown -R www-data:www-data /usr/src/mautic

# Copy init scripts and custom .htaccess
COPY docker-entrypoint.sh /entrypoint.sh
COPY makeconfig.php /makeconfig.php
COPY makedb.php /makedb.php
COPY mautic.crontab /etc/cron.d/mautic
COPY mautic-php.ini /usr/local/etc/php/conf.d/mautic-php.ini
# Enable Apache Rewrite Module
RUN a2enmod rewrite

ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
