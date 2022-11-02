---
sidebar_position: 1
title: PostgreSQL
description: "Setting up PostgreSQL on Datafold"
---
:::caution
Using Postgres **without** Cloudwatch will not be compatible with Column-level Lineage.
:::

**Steps to complete:**

1. [Run SQL Script for Permissions](postgres.md#run-sql-script)
2. [Configure your data source in Datafold](postgres.md#configure-in-datafold)

### Run SQL Script
To connect to Postgres, you need to create a user with read-only access to all tables in all schemas, write access to Datafold-specific schema for temporary tables:

```sql
/* Datafold utilizes a temporary dataset to materialize scratch work and keep data processing in the your warehouse. */

CREATE SCHEMA datafold_tmp;

/* Create a datafold user */

CREATE ROLE datafold WITH LOGIN ENCRYPTED PASSWORD 'SOMESECUREPASSWORD';

/* Give the datafole role write access to the temporary schema */

GRANT ALL ON SCHEMA datafold_tmp TO datafold;

/* Make sure that the postgres user has read permissions on the tables */

GRANT USAGE ON SCHEMA <myschema> TO datafold;
GRANT SELECT ON ALL TABLES IN SCHEMA <myschema> TO datafold;

```

### Configure in Datafold

| Field Name      | Description |
| ----------- | ----------- |
| Name     | A name given to the data source within Datafold |
| Host   | The hostname address for your database; default value `127.0.0.1` |
| Port   | Postgres connection port; default value is 5432 |
| User   | The user role created in our SQL script - `datafold`  |
| Password  | The password created in our SQL script |
| Database Name  | The name of the Postgres database you want to connect to |
| Schema for temporary tables  | The schema created in our SQL script - `datafold_tmp` |

Click **Create**. Your data source is ready!