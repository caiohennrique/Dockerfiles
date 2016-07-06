# ------------------------------------------------------------------------------
# Based image Developer Teknisa.
# ------------------------------------------------------------------------------
# Pull base image.

# ------------------------------------------------------------------------------
# Operational System

FROM debian
MAINTAINER Caio Rodrigues <caio.rodrigues@teknisa.com>
# ------------------------------------------------------------------------------
# Install base

RUN apt-get update && apt-get install -y \
        supervisor \
        build-essential \
        g++ \
        curl \
        libssl-dev \
        apache2-utils \
        git \
        libxml2-dev \
        wget \
        vim \
        sshfs \
        build-essential \
        unzip \
        libaio1 \
        sudo
# -----------------------------------------------------------------------------
# Create user developer

RUN groupadd -r developer && useradd -ms /bin/bash developer -g developer && sudo -u developer mkdir /home/developer/workfolder

# ------------------------------------------------------------------------------
# Install PHP/Apache2

RUN apt-get install -y apache2 php5 libapache2-mod-php5 php5-curl php-pear php5-dev php5-mcrypt && \
        echo "developer ALL = NOPASSWD: /etc/init.d/apache2" >> /etc/sudoers && \
        echo "umask 0002" >> /etc/apache2/ennvars && \
        chown -R developer.developer /etc/apache2/ && \
        chown -R developer.developer /var/log/apache2 && \
        ln -s /var/log/apache2/error.log /home/developer/workfolder/error.log && \
        /etc/init.d/apache2 restart && \
        printf "<Directory /home/developer/workfolder/>\nOptions +Indexes +FollowSymLinks +MultiViews\nAllowOverride All\nOrder allow,deny\nallow from All\nRequire all granted\n</Directory>" >> /etc/apache2/sites-available/000-default.conf && \
        sed -i 's/ \/var\/www\/html/ \/home\/developer\/workfolder/' /etc/apache2/sites-available/000-default.conf && \
        sed -i 's/display_errors = Off/display_errors = On/' /etc/php5/apache2/php.ini && \
        sed -i 's/display_errors = Off/display_errors = On/' /etc/php5/cli/php.ini


# ------------------------------------------------------------------------------
# Install Nodejs/Composer

RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - && \
        apt-get install -y nodejs && \
        curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer


# ------------------------------------------------------------------------------
# Install OracleClient and Oci8
COPY instantclient-basic-linux.x64-12.1.0.2.0.zip /opt
COPY instantclient-sdk-linux.x64-12.1.0.2.0.zip /opt

RUN mkdir /opt/oracle/ && mv /opt/instantclient-* /opt/oracle/ && \
        unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle && \
        unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle && \
        ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so && \
        ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so && \
        rm -Rf /opt/*.zip && \
        cd /tmp && wget https://pecl.php.net/get/oci8-2.0.11.tgz && \
        tar -xvf oci8-2.0.11.tgz && cd oci8-2.0.11 && \
        phpize && ./configure --with-oci8=shared,instantclient,/opt/oracle/instantclient_12_1 && \
        make && make install && \
        touch /etc/php5/mods-available/oci8.ini && \
        echo "extension = oci8.so"> /etc/php5/mods-available/oci8.ini && \
        php5enmod oci8

# ------------------------------------------------------------------------------------------------
# Install MongoExtension for PHP

RUN printf "\n" | pecl install mongo && \
        touch /etc/php5/mods-available/mongo.ini && \
        echo "extension=mongo.so" >> /etc/php5/mods-available/mongo.ini && \
        php5enmod mongo

# ------------------------------------------------------------------------------------------------
# Install Cloud9

RUN mkdir /opt/cloud9 && chown -R developer.developer /opt/cloud9

USER developer

RUN git config --global url."https://".insteadOf git:// && \
        git clone https://github.com/c9/core.git /opt/cloud9 && \
        /opt/cloud9/scripts/install-sdk.sh
        
#----------------------------------------------------------------------------------------------------
# Install Bower

USER root

RUN npm install -g bower && \
        echo "developer ALL = NOPASSWD: /usr/bin/npm" >> /etc/sudoers

#----------------------------------------------------------------------------------------------------
# Configurações finais

ADD confs/cloud9.conf /etc/supervisor/conf.d/
ADD confs/apache2.conf /etc/supervisor/conf.d/
RUN sed -i 's/^\(\[supervisord\]\)$/\1\nnodaemon=true/' /etc/supervisor/supervisord.conf && \
        apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR /home/developer/workfolder
EXPOSE 80 8181 8182 8183

CMD ["/usr/bin/supervisord"]

