# ClickHouse Server with FreeTDS and clickhouse-odbc-bridge

A custom ClickHouse server image, ready to connect to Microsoft SQL Server without using proprietary libraries. Based on the official linux/amd64 ClickHouse Server Docker Image.

See the [Dockerfile](./Dockerfile) for details.
[https://hub.docker.com/r/andreaskluth/clickhouse-server-sqlserver](https://hub.docker.com/r/andreaskluth/clickhouse-server-sqlserver
)

## odbc.ini and odbcinst.ini

FreeTDS needs an `odbc.ini` and an `odbcinst.ini` mounted to `/etc/odbc.ini` and `/etc/odbcinst.ini`. This image comes with a basic `/etc/odbcinst.ini` a `odbc.ini` must be provided by you.

A basic configuration would look like:

### **odbc.ini**

```text
[MyServer]
Driver = FreeTDS
Server = <<server-ip or dns-name>>
Port = 1433
# Or alternatively
# ServerName = <<needs to be configured in /etc/freetds/freetds.conf>>
```

### odbcinst.ini

```text
[FreeTDS]
Description=FreeTDS Driver for Linux & MSSQL
Driver=/usr/lib/x86_64-linux-gnu/odbc/libtdsodbc.so
UsageCount=1
```

The DSN configured in `odbc.ini` can be validated by issuing the following command in
the docker container. Replace `<<username>>` and `<<password>>` with an SQL
Server user that has permissions to connect to the database via TCP.

```bash
$ isql MyServer <<username>> <<password>>

+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> SELECT 1;
+------------+
|            |
+------------+
| 1          |
+------------+
SQLRowCount returns 1
1 rows fetched
SQL>
```

When the above command works, accessing the SQL Server from ClickHouse is as easy as:

```sql
# Validate the odbc bridge
SELECT * FROM odbc('DSN=dummy', dummy);

# Validate the sqlserver connection
SELECT * FROM odbc('DSN=MyServer;UID=MyUser;Pwd=Strong1.MyPassword;Database=MyDatabase', 'MyTable');


Query id: b64fbf5d-319f-40e9-a402-21f09a641a19

   ┌─ID─┬─VALUE─┐
1. │  1 │ one   │
2. │  2 │ value │
   └────┴───────┘

2 rows in set. Elapsed: 0.008 sec. 

```

## Debug Notes

ClickHouse starts the odbc-bridge with the following arguments:

```bash
clickhouse-odbc-bridge --http-port 9018 --listen-host 127.0.0.1 --http-timeout 30000000 --http-max-field-value-size 99999999999999999
```

## Developer Notes: Updating this image

### Deploy to docker-hub

```bash
docker build --build-arg TAG=25.4 -t andreaskluth/clickhouse-server-sqlserver:25.4 .
docker push andreaskluth/clickhouse-server-sqlserver:25.4
```
