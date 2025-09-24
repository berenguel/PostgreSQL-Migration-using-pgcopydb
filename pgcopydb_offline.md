# **Copying a database offline using pgcopydb**

This guide outlines the steps to copy a database from any PostgreSQL to any PostgreSQL offline.
I am focusing on Azure Database for PostgreSQL in this script, as you can see from Step 0

use case(s): create lower (test/dev) environments; migrate a database within a window


## Pre Requisites

### Step 0 - Role Privileges (Source & Target Servers)

```sql
GRANT azure_pg_admin TO demo;
```


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



### Step 4 - Clone the database offline

```sql
pgcopydb clone --skip-extensions --drop-if-exists --skip-ext-comments --no-owner --no-acl --filters /var/lib/postgresql/filter_schema.ini
```

### Step 5 - Clean up

```
pgcopydb stream cleanup && \
rm -rf ~/.local/share/pgcopydb/ && \
rm -rf /tmp/pgcopydb/*
```


### Step 6 - Compare schema and data

When using a filter file, 
```sql
pgcopydb compare schema
pgcopydb compare data


```
