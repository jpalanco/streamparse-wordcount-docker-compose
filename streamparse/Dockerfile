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

RUN apt-get install -y curl python-dev build-essential
RUN apt-get install -y python-pip
RUN apt-get install -y libssl-dev
RUN apt-get install -y libffi-dev 
RUN apt-get install -y wget 
RUN pip install -U pip


# download and install Leiningen
ENV LEIN_ROOT=1
RUN curl https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein > ./lein
RUN chmod a+x ./lein
RUN mv ./lein /usr/bin/lein
RUN lein version

RUN pip install streamparse

ENV STORM_USER=storm \
    STORM_CONF_DIR=/conf \
    STORM_DATA_DIR=/data \
    STORM_LOG_DIR=/logs

WORKDIR /opt

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


ENV PATH /opt/"$DISTRO_NAME"/bin/:$PATH

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

RUN apt-get install -y inetutils-ping vim
RUN update-ca-certificates -f
WORKDIR /tasks/wordcount

#ENTRYPOINT ["/bin/bash"]

