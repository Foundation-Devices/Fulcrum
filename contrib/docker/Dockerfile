FROM debian:trixie as builder

LABEL maintainer="Jean-Pierre De Jesus Diaz <jean@foundationdevices.com>, Dan Miller <dan@foundation.xyz>"
LABEL upstream.maintainer="Axel Gembe <derago@gmail.com>"
LABEL modification="System libraries version (no bundled binaries)"

ARG MAKEFLAGS

RUN apt update -y && \
    apt install -y openssl git build-essential pkg-config zlib1g-dev libbz2-dev libjemalloc-dev libzmq3-dev qtbase5-dev qt5-qmake librocksdb-dev cppzmq-dev 

WORKDIR /src

COPY . .

# Remove bundled libraries as we'll use system versions
RUN rm -rf src/Json/simdjson src/bitcoin/secp256k1 src/zmq staticlibs

RUN qmake -makefile PREFIX=/usr \
    CONFIG+=config_without_bundled_cppzmq \
    CONFIG+=config_without_bundled_secp256k1 \
    LIBS+=-lrocksdb \
    Fulcrum.pro && \
    make $MAKEFLAGS install

FROM debian:trixie-slim

RUN apt update && \
    apt install -y openssl libqt5network5 zlib1g libbz2-1.0 libjemalloc2 libzmq5 python3 librocksdb9.8 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder /src/Fulcrum /usr/bin/Fulcrum
COPY --from=builder /src/FulcrumAdmin /usr/bin/FulcrumAdmin

VOLUME ["/data"]
ENV DATA_DIR /data

ENV SSL_CERTFILE ${DATA_DIR}/fulcrum.crt
ENV SSL_KEYFILE ${DATA_DIR}/fulcrum.key

EXPOSE 50001 50002

COPY contrib/docker/docker-entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

CMD ["Fulcrum"]
