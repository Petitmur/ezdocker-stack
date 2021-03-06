FROM klabs/php56
MAINTAINER Kaliop

LABEL php.version=5.6 \
      application.type=ezpublish

ARG DOCKER_TIMEZONE=Europe/Paris
ARG DOCKER_LOCALE=fr_FR.UTF-8

# Configure timezone
# -----------------------------------------------------------------------------
RUN echo $DOCKER_TIMEZONE > /etc/timezone; dpkg-reconfigure -f noninteractive tzdata

# Base packages
# Java (used for eg. eZTika indexing)
# poppler-utils used for pdftotext binary (note that eztika is a much better option!)
# Mysql client, useful for db dumps and such
# @todo remove any unneeded ones (sudo, openssl?)
# -----------------------------------------------------------------------------
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    curl \
    wget \
    telnet \
    git \
    pwgen \
    locales \
    sudo \
    cron \
    vim \
    nano \
    mysql-client \
    imagemagick \
    poppler-utils \
    openjdk-7-jre-headless;

# Configure locale for date, time & currency php functions
# q: for what is the locale setup needed ?
# -----------------------------------------------------------------------------
RUN dpkg-reconfigure locales && \
    echo $DOCKER_LOCALE' UTF-8'>> /etc/locale.gen && \
    locale-gen $DOCKER_LOCALE && \
    /usr/sbin/update-locale LANG=$DOCKER_LOCALE

ENV LC_ALL $DOCKER_LOCALE
ENV LANG $DOCKER_LOCALE
ENV LANGUAGE $DOCKER_LOCALE

# Local user
# @todo simplify this as much as possible
# -----------------------------------------------------------------------------
ADD profile /tmp/profile

# nb: the 1013 used here for user id and group id is later on replaced by the code in bootstrap.sh...
RUN addgroup --gid 1013 site && \
    adduser --system --uid=1013 --gid=1013 \
        --home /home/site --shell /bin/bash site && \
    adduser site sudo && \
    adduser site site && \
    sed -i '$ a site   ALL=\(ALL:ALL\) NOPASSWD: ALL' /etc/sudoers; \
    mkdir -p /home/site/.ssh; \
    curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o /home/site/.git-completion.bash; \
    curl -o /home/site/.git-prompt.sh -OL https://github.com/git/git/raw/master/contrib/completion/git-prompt.sh; \
    cp /etc/skel/.* /home/site/; \
    cat /tmp/profile/.bashrc_append >> /home/site/.bashrc; \
    cat /tmp/profile/.vimrc_append >> /home/site/.vimrc; \
    cp /tmp/profile/.gitconfig /home/site/; \
    chown -R site:site /home/site;

# Misc
# @todo simplify this as much as possible, or plain remove
# Disabled for now : may be needed for some ssh connections inside container
# -----------------------------------------------------------------------------
#RUN echo 'StrictHostKeyChecking no' >> /etc/ssh/ssh_config;


# Composer global install
# -----------------------------------------------------------------------------
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin && \
    mv /usr/local/bin/composer.phar /usr/local/bin/composer && \
    chmod 755 /usr/local/bin/composer && \
    mkdir -p /home/site/.composer && \
    chown -R site:site /home/site/.composer


# -----------------------------------------------------------------------------

# Clear archives in apt cache folder
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

COPY bootstrap.sh /root/bootstrap.sh
RUN chmod 755 /root/bootstrap.sh

WORKDIR /var/www/

CMD ["/root/bootstrap.sh"]
