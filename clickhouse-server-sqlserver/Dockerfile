ARG TAG=head

FROM ubuntu:22.04 AS builder

# Install dependencies
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
    build-essential \
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
    apt-get clean

# Build FreeTDS from source
WORKDIR /build
RUN git clone https://github.com/FreeTDS/freetds.git 
WORKDIR /build/freetds
RUN git checkout v1.5.1 && \
    ./autogen.sh && \
    ./configure --prefix=/opt/freetds --with-openssl --enable-odbc && \
    make && \
    make install


FROM clickhouse/clickhouse-server:$TAG
ARG TAG


# Copy FreeTDS from builder stage
COPY --from=builder /opt/freetds /opt/freetds

# Set env for shared libraries
ENV LD_LIBRARY_PATH=/opt/freetds/lib:/usr/lib:/lib

# Install GPG and needed libs
RUN apt-get update && apt-get install -y --no-install-recommends gpg gpg-agent dirmngr unixodbc libct4 libsybdb5 && rm -rf /var/lib/apt/lists/*

## Add clickhouse-odbc-bridge
# Add keys
RUN GNUPGHOME=$(mktemp -d) && \
GNUPGHOME="$GNUPGHOME" gpg --no-default-keyring --keyring /usr/share/keyrings/clickhouse-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 8919F6BD2B48D754 && \
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/clickhouse-keyring.gpg] https://packages.clickhouse.com/deb stable main" | tee \/etc/apt/sources.list.d/clickhouse.list && \
apt-get update && apt-get install -y --no-install-recommends clickhouse-odbc-bridge && rm -rf /var/lib/apt/lists/*

## Add the basic driver config
COPY odbcinst.ini /etc/odbcinst.ini
