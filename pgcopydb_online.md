# **Copying a database online using pgcopydb**

This guide outlines the steps to copy a database from any PostgreSQL to any PostgreSQL online.
I am focusing on Azure Database for PostgreSQL in this script, as you can see from the permissions

use case(s): migrate a database with near zero downtime; upgrade a PostgreSQL database with near zero downtime


## Pre Requisites

### Step 0 - Role Privileges & Server parameters

Note that this must be done on source & target servers

```sql
ALTER ROLE demo WITH REPLICATION;
GRANT azure_pg_admin TO demo;
```
For full user configuration, please refer to this resource [Configuring a replication user](https://github.com/berenguel/bi-directional-replication-in-Flexible-Server/blob/main/configuring_replication_user.sql)

```sql
Wal_level=logical
Max_worker_processes=16
Max_replication_slots=10
Max_wal_senders=10
Track_commit_timestamp=ON
```

This process is based on logical replication. Therefore, be aware of the following requirement:
- Tables must have a primary key or unique key, or REPLICA IDENTITY must be set to FULL.



### Step 1 - Prepare the Azure Database for PostgreSQL URIs

```sql
export PGCOPYDB_SOURCE_PGURI="postgres://<user>:<password>@<hostname>:<port>/<database>"
export PGCOPYDB_TARGET_PGURI="postgres://<user>:<password>@<hostname>:<port>/<database>"
```

### Step 2 - Health check - URIs & connectivity to the database servers

```sql
pgcopydb ping

```

### Step 3 - Prepare a filter file [optional]
```sql
vi /var/lib/postgresql/filter_schema.ini

[exclude-schema]
pg_catalog
information_schema

```

### Step 4 - Clone the database online

```sql
pgcopydb clone --follow --table-jobs 8 --index-jobs 8 --restore-jobs 8 --skip-extensions --skip-ext-comments  --no-owner  --no-acl --skip-db-properties  --drop-if-exists &

```

### Step 5 - Application is ready to make the switch

```sql
pgcopydb stream sentinel set endpos --current
```


### Step 6 - Clean up

```
pgcopydb stream cleanup && \
rm -rf ~/.local/share/pgcopydb/ && \
rm -rf /tmp/pgcopydb/*
```


### Step 7 - Compare schema and data

```sql
pgcopydb compare schema --notice
pgcopydb compare data


```

Note that if you are using a filter file, **pgcopydb compare data** will fail with a message similar to this:
```
16:02:56.949 16981 INFO   Current filtering setup is: {"type":"SOURCE_FILTER_TYPE_NONE"}
16:02:56.949 16981 INFO   Catalog filtering setup is: {"type":"SOURCE_FILTER_TYPE_EXCL","exclude-schema":...}
```
