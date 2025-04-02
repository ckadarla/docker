### ================================== ###
###   STAGE 1 CREATE PARENT IMAGE      ###
### ================================== ###

# https://www.docker.com/blog/docker-arm-virtual-meetup-multi-arch-with-buildx/

FROM alpine:3.12 

ARG TARGETPLATFORM
ARG BUILDPLATFORM


ENV NAGIOS_HOME            /opt/nagios
ENV NAGIOS_USER            nagios
ENV NAGIOS_GROUP           nagios
ENV NAGIOS_CMDUSER         nagios
ENV NAGIOS_CMDGROUP        nagios
ENV NAGIOS_FQDN            nagios.example.com
ENV NAGIOSADMIN_USER       nagiosadmin
ENV NAGIOSADMIN_PASS       nagios
ENV APACHE_RUN_USER        nagios
ENV APACHE_RUN_GROUP       nagios
ENV NAGIOS_TIMEZONE        UTC
ENV DEBIAN_FRONTEND        noninteractive
ENV NG_NAGIOS_CONFIG_FILE  ${NAGIOS_HOME}/etc/nagios.cfg
ENV NG_CGI_DIR             ${NAGIOS_HOME}/sbin
ENV NG_WWW_DIR             ${NAGIOS_HOME}/share/nagiosgraph
ENV NG_CGI_URL             /cgi-bin
ENV NAGIOS_BRANCH          nagios-4.5.7
ENV NAGIOS_PLUGINS_BRANCH  release-2.4.12
ENV NRPE_BRANCH            nrpe-4.1.1
ENV NCPA_BRANCH            v3.1.1
ENV NSCA_BRANCH            nsca-2.10.2
ENV NAGIOSTV_VERSION       0.9.2


ENV NAGIOS_HOME=/opt/nagios \
    NAGIOS_USER=nagios \
    NAGIOS_GROUP=nagios \
    NAGIOS_CMDUSER=nagios \
    NAGIOS_CMDGROUP=nagios \
    NAGIOS_TIMEZONE=UTC \
    NAGIOS_FQDN=nagios.example.com \
    NAGIOSADMIN_USER=nagiosadmin \
    NAGIOSADMIN_PASS=nagiosadmin \
    NAGIOS_VERSION=4.5.9 \
    NAGIOS_PLUGINS_VERSION=2.4.12 \
    NRPE_VERSION=4.1.3 \
    APACHE_LOCK_DIR=/var/run \
    APACHE_LOG_DIR=/var/log/apache2
ENV nagiosgraphversion=1.5.2


RUN addgroup -S ${NAGIOS_GROUP} && \
    adduser  -S ${NAGIOS_USER} -G ${NAGIOS_CMDGROUP} -g ${NAGIOS_USER} && \
    apk update && \
    apk add --no-cache git curl unzip apache2 apache2-utils rsyslog \
                        php7 php7-gd php7-cli runit parallel ssmtp \
                        libltdl libintl openssl-dev php7-apache2 procps tzdata \
                        libldap mariadb-connector-c freeradius-client-dev libpq libdbi \
                        lm-sensors perl net-snmp-perl perl-net-snmp perl-crypt-x509 \
                        perl-timedate perl-libwww perl-text-glob samba-client openssh openssl \
                        net-snmp-tools bind-tools gd gd-dev && \
                                                \
    : '# For x86 the binary is : gosu-i386' && \
    : '# For x64 the binary is : gosu-amd64' && \
    : '# For arm-v6 the binary is : gosu-armel' && \
    : '# For arm-v7 the binary is : gosu-armhf' && \
    : '# For arm64 the binary is : gosu-arm64' && \
    : '# For arm64/v8 the binary is : gosu-arm64' && \
    : '#######################################' && \
    : '# Creating an associative array with the platforms and their respective gosu release DOES NOT WORK in /bin/sh' && \
    echo "Arguments TARGETPLATFORM: ${TARGETPLATFORM} and BUILDPLATFORM: ${BUILDPLATFORM}" && \
    echo "$TARGETPLATFORM" | awk '{ gosuBinArr["linux/386"]="gosu-i386"; gosuBinArr["linux/amd64"]="gosu-amd64"; gosuBinArr["linux/arm/v6"]="gosu-armel"; gosuBinArr["linux/arm/v7"]="gosu-armhf"; gosuBinArr["linux/arm64"]="gosu-arm64"; gosuBinArr["linux/arm64/v8"]="gosu-arm64"; print gosuBinArr[$0];}' > mygosuver.txt && \
    gosuPlatform=$(cat mygosuver.txt) && \
    echo "Downloading ${gosuPlatform} for platform $TARGETPLATFORM" &&\
    curl -L -o gosu "https://github.com/tianon/gosu/releases/download/1.17/${gosuPlatform}"  && \
    mv gosu /bin/ && \
    chmod 755 /bin/gosu && \
    addgroup -S apache ${NAGIOS_CMDGROUP}


### ================================== ###
###   STAGE 2 COMPILE NAGIOS SOURCES   ###
### ================================== ###

#FROM builder-base as builder-compile
ARG TARGETPLATFORM
ARG BUILDPLATFORM

# Add dependencies required to build Nagios
RUN apk update && \
    apk add --no-cache build-base automake libtool autoconf py-docutils gnutls  \
                        gnutls-dev g++ make alpine-sdk build-base gcc autoconf \
                        gettext-dev linux-headers openssl-dev net-snmp net-snmp-tools \
                        libcrypto1.1 libpq musl libldap libssl1.1 libdbi freeradius-client mariadb-connector-c \
                        openssh-client bind-tools samba-client fping grep rpcbind \
                        lm-sensors net-snmp-tools \
                        file freeradius-client-dev libdbi-dev libpq linux-headers mariadb-dev \
                        mariadb-connector-c-dev perl \
                        net-snmp-dev openldap-dev openssl-dev postgresql-dev wget


RUN cd /tmp                                                          && \
    git clone https://git.code.sf.net/p/nagiosgraph/git nagiosgraph  && \
    cd nagiosgraph                                                   && \
    ./install.pl --install                                      \
        --prefix /opt/nagiosgraph                               \
        --nagios-user ${NAGIOS_USER}                            \
        --www-user ${NAGIOS_USER}                               \
        --nagios-perfdata-file ${NAGIOS_HOME}/var/perfdata.log  \
        --nagios-cgi-url /cgi-bin                               \
                                                                     && \
#    cp share/nagiosgraph.ssi ${NAGIOS_HOME}/share/ssi/common-header.ssi && \
#    cd /tmp && rm -Rf nagiosgraph
