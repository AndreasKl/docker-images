ARG TAG=head

FROM ubuntu:22.04 AS builder

ARG DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    git \
    autoconf \
    automake \
    libtool \
    pkg-config \
    unixodbc-dev \
    libssl-dev \
    libiconv-hook-dev \
    gettext \
    autopoint \
    m4 \
    gperf \
    curl && \
    rm -rf /var/lib/apt/lists/* /var/cache/debconf /tmp/*

# Build FreeTDS from source
WORKDIR /build
RUN git clone https://github.com/FreeTDS/freetds.git 
WORKDIR /build/freetds

# Checkout v1.5.1 via the commit hash and build freetds
RUN git checkout 349445e && \
    ./autogen.sh && \
    ./configure --prefix=/opt/freetds --with-openssl --enable-odbc && \
    make && \
    make install


FROM clickhouse/clickhouse-server:$TAG
ARG TAG

# Copy FreeTDS from builder stage
COPY --from=builder /opt/freetds /opt/freetds

# Install GPG and needed libs
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gpg gpg-agent dirmngr unixodbc libct4 libsybdb5 && \
    rm -rf /var/lib/apt/lists/* /var/cache/debconf /tmp/*

## Add clickhouse-odbc-bridge
# Add keys
RUN GNUPGHOME=$(mktemp -d) && \
    GNUPGHOME="$GNUPGHOME" gpg --no-default-keyring --keyring /usr/share/keyrings/clickhouse-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 8919F6BD2B48D754 && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/clickhouse-keyring.gpg] https://packages.clickhouse.com/deb stable main" | tee \/etc/apt/sources.list.d/clickhouse.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    clickhouse-odbc-bridge && \
    rm -rf /var/lib/apt/lists/* /var/cache/debconf /tmp/*

## Add the basic driver config
COPY odbcinst.ini /etc/odbcinst.ini
