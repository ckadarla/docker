FROM debian:latest

# Set environment variables
ENV NAGIOS_VERSION=4.5.9 \
    NAGIOS_PLUGINS_VERSION=2.4.6 \
    NAGIOS_USER=nagios \
    NAGIOS_GROUP=nagcmd \
    NAGIOSGRAPH_VERSION=1.5.2

# Install required packages
RUN apt-get update && apt-get install -y \
    apache2 \
    build-essential \
    libgd-dev \
    libapache2-mod-php \
    php \
    unzip \
    wget \
    curl \
    perl \
    libperl-dev \
    libssl-dev \
    libpng-dev \
    librrds-perl \
    libgd-perl \
    libgraphviz-perl \
    rrdtool \
    libapache2-mod-perl2 \
    && rm -rf /var/lib/apt/lists/*

# Create Nagios user and group
RUN useradd -m -s /bin/bash ${NAGIOS_USER} && \
    groupadd ${NAGIOS_GROUP} && \
    usermod -a -G ${NAGIOS_GROUP} ${NAGIOS_USER}

# Download and install Nagios Core
WORKDIR /tmp
RUN wget https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-${NAGIOS_VERSION}/nagios-${NAGIOS_VERSION}.tar.gz && \
    tar -xzf nagios-${NAGIOS_VERSION}.tar.gz && \
    cd nagios-${NAGIOS_VERSION} && \
    ./configure --with-nagios-group=${NAGIOS_GROUP} --with-command-group=${NAGIOS_GROUP} && \
    make all && \
    make install && \
    make install-init && \
    make install-commandmode && \
    make install-config && \
    make install-webconf && \
    usermod -a -G ${NAGIOS_GROUP} www-data

# Install Nagios Plugins
RUN wget https://nagios-plugins.org/download/nagios-plugins-${NAGIOS_PLUGINS_VERSION}.tar.gz && \
    tar -xzf nagios-plugins-${NAGIOS_PLUGINS_VERSION}.tar.gz && \
    cd nagios-plugins-${NAGIOS_PLUGINS_VERSION} && \
    ./configure --with-nagios-user=${NAGIOS_USER} --with-nagios-group=${NAGIOS_GROUP} && \
    make && make install

# Download and Install NagiosGraph
RUN wget -O nagiosgraph-${NAGIOSGRAPH_VERSION}.tar.gz "https://downloads.sourceforge.net/project/nagiosgraph/nagiosgraph/1.5.2/nagiosgraph-1.5.2.tar.gz" && \
    tar -xzf nagiosgraph-${NAGIOSGRAPH_VERSION}.tar.gz && \
    cd nagiosgraph-${NAGIOSGRAPH_VERSION} && \
    chmod +x install.pl && \
    ls -lah && \
    perl install.pl --check-prereq 


# Configure Apache
RUN a2enmod cgi && systemctl restart apache2

# Expose ports
EXPOSE 80

# Start services
CMD ["/usr/local/nagios/bin/nagios", "/usr/local/nagios/etc/nagios.cfg"]
