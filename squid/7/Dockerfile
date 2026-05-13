FROM docker.io/ubuntu:26.04

LABEL org.opencontainers.image.title="Squid 7 Proxy" \
      org.opencontainers.image.description="Squid 7 Proxy image built on Ubuntu 24.04" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.url="https://github.com/landregistry/docker-base-images/blob/master/README.md" \
      org.opencontainers.image.source="https://github.com/landregistry/docker-base-images" \
      org.opencontainers.image.vendor="HM Land Registry"

ARG TARGETARCH
ARG SQUID_RELEASE=SQUID_7_5
ARG SQUID_VERSION=squid-7.5

ENV SQUID_CACHE_DIR=/var/spool/squid \
    SQUID_LOG_DIR=/var/log/squid \
    SQUID_SSLDB_DIR=/var/lib/ssl_db \
    SQUID_USER=proxy \
    SQUID_GROUP=root

WORKDIR /tmp

RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get -qq install -y --no-install-recommends \
      ca-certificates=20260223 \
      g++=4:15.2.0-5ubuntu1 \
      make=4.4.1-3 \
      libcrypt-dev=1:4.5.1-1 \
      libssl-dev=3.5.5-1ubuntu3 \
      pkg-config=2.5.1-4 \
      wget=1.25.0-2ubuntu4 \
 && rm -rf /var/lib/apt/lists/* \
 && update-ca-certificates -f \
 && wget -q https://github.com/squid-cache/squid/releases/download/${SQUID_RELEASE}/${SQUID_VERSION}.tar.gz \
 && tar xzf ${SQUID_VERSION}.tar.gz \
 && ( \
      cd ${SQUID_VERSION} && \
      CGFLAGS="-g0 -O2" \
      CXXFLAGS="-g0 -O2" \
      LDFLAGS="-s" \
      ./configure --enable-ssl-crtd --with-openssl --prefix=/usr --localstatedir=/var \
          --libexecdir="/usr/lib/squid" --datadir="/usr/share/squid" \
          --sysconfdir=/etc/squid --with-default-user=proxy --with-logdir=/var/log/squid \
          --with-pidfile=/var/run/squid.pid --build="$(uname -m)" --host="$(uname -m)" \
          --disable-arch-native && \
      make -s install \
    ) \
 && rm -rf ${SQUID_VERSION} && rm ${SQUID_VERSION}.tar.gz \
 && wget -q https://github.com/krallin/tini/releases/download/v0.19.0/tini-${TARGETARCH} -O /tini \
 && chmod +x /tini \
 && usermod -a -G ${SQUID_GROUP} ${SQUID_USER}

COPY squid.conf ssl.pem entrypoint.sh /etc/squid/

EXPOSE 3128/tcp
VOLUME /var/spool/squid

# Set Tini entrypoint
ENTRYPOINT ["/tini", "-e", "143", "--"]

# Set squid CMD
CMD ["bash", "/etc/squid/entrypoint.sh"]
