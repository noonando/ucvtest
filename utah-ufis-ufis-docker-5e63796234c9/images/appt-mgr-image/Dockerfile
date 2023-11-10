FROM ubuntu:18.04

ENV DEBIAN_FRONTEND=noninteractive


COPY ./oracle-client /tmp

RUN apt-get update -yqq && apt-get install software-properties-common -yq && add-apt-repository ppa:ondrej/php -y && apt-get install -yq --no-install-recommends \
    apt-utils \
    curl \
    # Install git
    git \
    # Install apache
    apache2 \
    # Install php 7.3
    php7.3 \
    libapache2-mod-php7.3 \
    php7.3-cli \
    php7.3-json \
    php7.3-curl \
    php7.3-fpm \
    php7.3-dev \
    php7.3-gd \
    php7.3-ldap \
    php7.3-mbstring \
    php7.3-bcmath \
    php7.3-mysql \
    php7.3-soap \
    php7.3-sqlite3 \
    php7.3-xml \
    php7.3-zip \
    php7.3-intl \
    libldap2-dev \
    libaio1 \
    libaio-dev \
    # Install tools
    openssl \
    nano \
    ghostscript \
    iputils-ping \
    locales \
    rlwrap \
    php-pear \
    make \
    unzip \
    zip \
    tar \
    vim \
    ca-certificates \
    && apt-get clean &&\
    # Install Oracle Client
    cd /opt &&\
    mkdir oracle &&\
    mv /tmp/instantclient* /opt/oracle/ &&\
    cd /opt/oracle/ &&\
    unzip instantclient-basic-linux.x64-12.2.0.1.0.zip &&\
    unzip instantclient-sqlplus-linux.x64-12.2.0.1.0.zip &&\
    unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip &&\
    cd /opt/oracle/instantclient_12_2/ &&\
    ln -s libclntsh.so.12.1 libclntsh.so &&\
    echo "/opt/oracle/instantclient_12_2/" >> /etc/ld.so.conf.d/oracle.conf &&\
    ldconfig &&\
    echo 'export ORACLE_HOME=/opt/oracle' >> ~/.bashrc &&\
    echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/oracle/instantclient_12_2' >> ~/.bashrc &&\
    echo 'PATH=$PATH:/opt/oracle/instantclient_12_2' >> ~/.bashrc &&\
    echo "alias sqlplus='/usr/bin/rlwrap -m /opt/oracle/instantclient_12_2/sqlplus'" >> ~/.bashrc &&\
    cd /opt/oracle &&\
    pecl download oci8-2.2.0 &&\
    tar -xzvf oci8*.tgz &&\
    cd oci8-2.2.0 &&\
    phpize &&\
    ./configure --with-oci8=instantclient,/opt/oracle/instantclient_12_2/ &&\
    make install &&\
    echo 'instantclient,/opt/oracle/instantclient_12_2' | pecl install oci8-2.2.0 &&\
    echo "extension=oci8.so" >> /etc/php/7.3/fpm/php.ini &&\
    echo "extension=oci8.so" >> /etc/php/7.3/cli/php.ini &&\
    echo "extension=oci8.so" >> /etc/php/7.3/apache2/php.ini &&\
    printf "\n" | pecl install xdebug redis &&\
    echo "zend_extension=/usr/lib/php/20180731/xdebug.so" >> /etc/php/7.3/fpm/php.ini &&\
    echo "zend_extension=/usr/lib/php/20180731/xdebug.so" >> /etc/php/7.3/cli/php.ini &&\
    echo "zend_extension=/usr/lib/php/20180731/xdebug.so" >> /etc/php/7.3/apache2/php.ini &&\
    echo "extension=redis.so" >> /etc/php/7.3/fpm/php.ini &&\
    echo "extension=redis.so" >> /etc/php/7.3/cli/php.ini &&\
    echo "extension=redis.so" >> /etc/php/7.3/apache2/php.ini

#Install xdebug
RUN pecl install xdebug-3.1.5
COPY xdebug.ini /etc/php/7.3/mods-available/
RUN phpenmod xdebug

# Install Node and NPM
ENV NODE_VERSION=15.5.0
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
ENV NVM_DIR=/root/.nvm
RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm use v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm alias default v${NODE_VERSION}
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"
RUN node --version
RUN npm --version

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set locales
RUN locale-gen en_US.UTF-8 en_GB.UTF-8 de_DE.UTF-8 es_ES.UTF-8 fr_FR.UTF-8 it_IT.UTF-8 km_KH sv_SE.UTF-8 fi_FI.UTF-8

# Configure PHP for Appt-Mgr
COPY ./appt-mgr.ini /etc/php/7.3/mods-available/
RUN phpenmod appt-mgr

# Configure apache for Appt-Mgr
RUN a2enmod rewrite expires proxy_http
RUN echo "ServerName localhost" | tee /etc/apache2/conf-available/servername.conf
RUN a2enconf servername

# Configure vhost for app-mgr
ENV APACHE_LOG_DIR=/var/log/apache2
RUN touch $APACHE_LOG_DIR/error.log
RUN touch $APACHE_LOG_DIR/access.log
COPY appt-mgr-vhost.conf /etc/apache2/sites-available/
RUN a2dissite 000-default
RUN a2ensite appt-mgr-vhost.conf

EXPOSE 80 443

WORKDIR /var/www

CMD apachectl -D FOREGROUND
