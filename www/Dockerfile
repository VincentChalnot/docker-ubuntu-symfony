FROM ubuntu:xenial

RUN ln -fs /usr/share/zoneinfo/Europe/Paris /etc/localtime

ENV DEBIAN_FRONTEND noninteractive
ENV WWW_HOME /var/www
RUN apt-get clean && \
    apt-get update && \
    apt-get -y dist-upgrade

# Install base packages
RUN apt-get install -y nano curl git wget sudo zsh locales apt-utils

# Rebuild locales properly
RUN locale-gen en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# Make default unix user id match www-data
RUN usermod -u 1000 www-data
# Allow shell access for www-data
RUN chsh -s /bin/zsh www-data
# Make www-data sudoer (without password)
RUN echo "www-data ALL=NOPASSWD: ALL" | (EDITOR="tee -a" visudo)
# Create home directory and set proper permissions
RUN mkdir -p $WWW_HOME
RUN chown www-data:1000 $WWW_HOME

USER www-data
RUN git clone --depth=1 https://github.com/robbyrussell/oh-my-zsh.git $WWW_HOME/.oh-my-zsh
COPY zshrc $WWW_HOME/.zshrc


### FROM THERE THE CONFIGURATION IS NOT GENERIC ANYMORE
USER root

# Custom php extensions
RUN apt-get install -y \
    mariadb-client \
    apache2 \
    libapache2-mod-php \
    php-cli \
    php-intl \
    php-xml \
    php-zip \
    php-mbstring \
    php-gd \
    php-apcu \
    php-mcrypt \
    php-mysql \
    php-curl

RUN apt-get install -y \
    bundler \
    ruby-dev \
    npm

RUN npm install -g uglifyjs uglifycss

# And because mcrypt does not get enabled by default:
RUN phpenmod mcrypt

# Enabling our custom config
COPY php/mods-available/symfony.ini /etc/php/7.0/mods-available/symfony.ini
RUN phpenmod symfony

# Enable apache mod rewrite
RUN a2enmod rewrite


# Install composer
ENV COMPOSER_ALLOW_SUPERUSER 1
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# DEV ONLY !!!
RUN apt-get install -y php-xdebug

# Rollback default shell
USER www-data

ENTRYPOINT ["sudo", "apachectl", "-DFOREGROUND"]
