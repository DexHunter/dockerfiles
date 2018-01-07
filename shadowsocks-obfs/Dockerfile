FROM alpine:latest

ARG SS_VER=3.1.2
ARG SS_URL=https://github.com/shadowsocks/shadowsocks-libev/releases/download/v$SS_VER/shadowsocks-libev-$SS_VER.tar.gz
ARG OBFS_VER=0.0.5
ARG OBFS_REPO=https://github.com/shadowsocks/simple-obfs

RUN set -ex && \
    apk add --no-cache --virtual .build-deps \
                                autoconf \
                                automake \
                                build-base \
                                curl \
                                libev-dev \
                                linux-headers \
                                libsodium-dev \
                                libtool \
                                mbedtls-dev \
                                pcre-dev \
                                tar \
                                c-ares-dev \
                                zlib-dev \
                                xmlto \
                                git

RUN set -ex && \
    cd /tmp && \
    curl -sSL $SS_URL | tar xz --strip 1 && \
    ./configure --prefix=/usr --disable-documentation && \
    make install && \
    cd / && \
    rm -rf /tmp/*

RUN set -ex && \
    cd /tmp && \
    git clone $OBFS_REPO -b v$OBFS_VER && cd simple-obfs && \
    git submodule update --init --recursive && \
    ./autogen.sh && \
    ./configure --prefix=/usr --disable-documentation && \
    make install && \
    cd / && \
    rm -rf /tmp/*

RUN set -ex & \
    runDeps="$( \
        scanelf --needed --nobanner /usr/bin/ss-* /usr/bin/obfs-* \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | xargs -r apk info --installed \
            | sort -u \
    )" && \
    apk add --no-cache --virtual .run-deps $runDeps && \
    apk del .build-deps

USER nobody
VOLUME ["/mnt/config.json"]
CMD ss-server -c "/mnt/config.json"
