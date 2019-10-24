FROM php:7.4-rc-apache

LABEL maintainer="alexcheng1982@gmail.com"
LABEL php_version="7.2.12"
LABEL magento_version="2.3.2"
LABEL description="Magento 2.3.2 with PHP 7.2.12"

ENV MAGENTO_VERSION 2.3.2
ENV INSTALL_DIR /var/www/html
ENV COMPOSER_HOME /var/www/.composer/

RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer
COPY ./auth.json $COMPOSER_HOME

RUN docker-php-ext-install pdo_mysql \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install zip \
    && docker-php-ext-install intl \
    && docker-php-ext-install xsl \
    && docker-php-ext-install soap \
    && docker-php-ext-install bcmath

# Install sodium latest
#RUN apt-get purge libsodium1.*
#RUN wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
#RUN tar xvzf LATEST.tar.gz
#RUN cd libsodium-stable && \
#    ./configure && \
#    make && make check && \
#    make install
#RUN docker-php-ext-disable
#RUN cd ..

RUN apt-get update \
    && apt-get install -y libmcrypt-dev \
    && yes '' | pecl install mcrypt-1.0.1 \
    && echo 'extension=mcrypt.so' > /usr/local/etc/php/conf.d/mcrypt.ini

RUN chsh -s /bin/bash www-data
RUN chown -R www-data:www-data /var/www
RUN su www-data -c "composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition $INSTALL_DIR $MAGENTO_VERSION"

RUN cd $INSTALL_DIR \
    && find . -type d -exec chmod 770 {} \; \
    && find . -type f -exec chmod 660 {} \; \
    && chmod u+x bin/magento

COPY ./install-magento /usr/local/bin/install-magento
RUN chmod +x /usr/local/bin/install-magento

COPY ./install-sampledata /usr/local/bin/install-sampledata
RUN chmod +x /usr/local/bin/install-sampledata

RUN a2enmod rewrite
RUN echo "memory_limit=2048M" > /usr/local/etc/php/conf.d/memory-limit.ini

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR $INSTALL_DIR

# Add cron job
#ADD crontab /etc/cron.d/magento2-cron
#RUN chmod 0644 /etc/cron.d/magento2-cron \
#    && crontab -u www-data /etc/cron.d/magento2-cron