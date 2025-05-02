ARG TAG=head

FROM clickhouse/clickhouse-server:$TAG
ARG TAG

# Install GPG and FreeDTS
RUN apt-get update && apt-get install -y --no-install-recommends gpg gpg-agent dirmngr unixodbc freetds-bin freetds-common freetds-dev libct4 libsybdb5 tdsodbc && rm -rf /var/lib/apt/lists/*

## Add clickhouse-odbc-bridge
# Add keys
RUN GNUPGHOME=$(mktemp -d) && \
GNUPGHOME="$GNUPGHOME" gpg --no-default-keyring --keyring /usr/share/keyrings/clickhouse-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 8919F6BD2B48D754 && \
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/clickhouse-keyring.gpg] https://packages.clickhouse.com/deb stable main" | tee \/etc/apt/sources.list.d/clickhouse.list && \
apt-get update && apt-get install -y --no-install-recommends clickhouse-odbc-bridge && rm -rf /var/lib/apt/lists/*

## Add the basic driver config
COPY odbcinst.ini /etc/odbcinst.ini
