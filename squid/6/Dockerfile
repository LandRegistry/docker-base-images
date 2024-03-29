FROM ubuntu:22.04

ARG SQUID_VERSION=6.7

ENV SQUID_CACHE_DIR=/var/spool/squid \
    SQUID_LOG_DIR=/var/log/squid \
    SQUID_SSLDB_DIR=/var/lib/ssl_db \
    SQUID_USER=proxy \
    SQUID_GROUP=root

RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y -q curl g++ make libssl-dev pkg-config \
 && rm -rf /var/lib/apt/lists/* \
 && curl -O http://www.squid-cache.org/Versions/v6/squid-${SQUID_VERSION}.tar.gz \
 && tar xzf squid-${SQUID_VERSION}.tar.gz \
 && cd squid-${SQUID_VERSION} \
 && ./configure --enable-ssl-crtd --with-openssl --prefix=/usr --localstatedir=/var --libexecdir=\${prefix}/lib/squid --datadir=\${prefix}/share/squid --sysconfdir=/etc/squid --with-default-user=proxy --with-logdir=/var/log/squid --with-pidfile=/var/run/squid.pid \
 && make install \
 && cd .. && rm -rf squid-${SQUID_VERSION} && rm squid-${SQUID_VERSION}.tar.gz \
 && curl -L https://github.com/krallin/tini/releases/download/v0.19.0/tini > /tini \
 && chmod +x /tini \
 && usermod -a -G ${SQUID_GROUP} ${SQUID_USER}

COPY squid.conf ssl.pem entrypoint.sh /etc/squid/

EXPOSE 3128/tcp
VOLUME /var/spool/squid

# Set Tini entrypoint
ENTRYPOINT ["/tini", "-e", "143", "--"]

# Set squid CMD
CMD ["bash", "/etc/squid/entrypoint.sh"]
