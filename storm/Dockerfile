FROM ubuntu:15.10

MAINTAINER Jose Ramon Palanco <jose.palanco@dinoflux.com>

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install openjdk-8-jre-headless -y && \
    apt-get install locales -y && \
    update-locale LANG=C.UTF-8 LC_MESSAGES=POSIX && \
    locale-gen en_US.UTF-8 && \
    dpkg-reconfigure locales && \
    apt-get clean all 

RUN apt-get install -y python openssh-server gnupg

ENV STORM_USER=storm \
    STORM_CONF_DIR=/conf \
    STORM_DATA_DIR=/data \
    STORM_LOG_DIR=/logs

# Add a user and make dirs
RUN set -x \
    && useradd  "$STORM_USER" \
    && mkdir -p "$STORM_CONF_DIR" "$STORM_DATA_DIR" "$STORM_LOG_DIR" \
    && chown -R "$STORM_USER:$STORM_USER" "$STORM_CONF_DIR" "$STORM_DATA_DIR" "$STORM_LOG_DIR"

ARG GPG_KEY=ACEFE18DD2322E1E84587A148DE03962E80B8FFD
ARG DISTRO_NAME=apache-storm-1.0.2

# Download Apache Storm, verify its PGP signature, untar and clean up
RUN set -x \
    && wget -q "http://www.apache.org/dist/storm/$DISTRO_NAME/$DISTRO_NAME.tar.gz" \
    && wget -q "http://www.apache.org/dist/storm/$DISTRO_NAME/$DISTRO_NAME.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$GPG_KEY" \
    && gpg --batch --verify "$DISTRO_NAME.tar.gz.asc" "$DISTRO_NAME.tar.gz" \
    && tar -xzf "$DISTRO_NAME.tar.gz" \
    && chown -R "$STORM_USER:$STORM_USER" "$DISTRO_NAME" \
    && rm -r "$GNUPGHOME" "$DISTRO_NAME.tar.gz" "$DISTRO_NAME.tar.gz.asc" 

WORKDIR $DISTRO_NAME

ENV PATH $PATH:/$DISTRO_NAME/bin

RUN wget https://github.com/javabean/su-exec/releases/download/v0.2/su-exec.amd64 -O /usr/bin/su-exec
RUN chmod +x /usr/bin/su-exec

#
# FIXME: streamparse should be installed by virtualenv
#
RUN apt-get install -y python-pip python-dev libffi-dev libssl-dev 
RUN pip install streamparse

COPY docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

WORKDIR /

ENTRYPOINT ["/docker-entrypoint.sh"]
