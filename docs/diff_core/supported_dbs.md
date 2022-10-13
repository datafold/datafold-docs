---
sidebar_position: 5
title: Supported Databases
---

| Database      | Connection string                                                                                                                   | Status |
|---------------|-------------------------------------------------------------------------------------------------------------------------------------|--------|
| PostgreSQL >=10    | `postgresql://<user>:<password>@<host>:5432/<database>`                                                                             |  💚    |
| MySQL         | `mysql://<user>:<password>@<hostname>:5432/<database>`                                                                              |  💚    |
| Snowflake     | With password:`"snowflake://<USER>:<password>@<ACCOUNT>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>"`, With SSO: `"snowflake://<USER>@<ACCOUNT>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>&authenticator=externalbrowser"` |  💚    |
| BigQuery      | `bigquery://<project>/<dataset>`                                                                                                    |  💚    |
| Redshift      | `redshift://<username>:<password>@<hostname>:5439/<database>`                                                                       |  💚    |
| Oracle        | `oracle://<username>:<password>@<hostname>/database`                                                                                |  💛    |
| Presto        | `presto://<username>:<password>@<hostname>:8080/<database>`                                                                         |  💛    |
| Databricks    | `databricks://<http_path>:<access_token>@<server_hostname>/<catalog>/<schema>`                                                      |  💛    |
| Trino         | `trino://<username>:<password>@<hostname>:8080/<database>`                                                                          |  💛    |
| Clickhouse    | `clickhouse://<username>:<password>@<hostname>:9000/<database>`                                                                     |  💛    |
| Vertica       | `vertica://<username>:<password>@<hostname>:5433/<database>`                                                                        |  💛    |
| ElasticSearch |                                                                                                                                     |  📝    |
| Planetscale   |                                                                                                                                     |  📝    |
| Pinot         |                                                                                                                                     |  📝    |
| Druid         |                                                                                                                                     |  📝    |
| Kafka         |                                                                                                                                     |  📝    |
| DuckDB        |                                                                                                                                     |  📝    |
| SQLite        |                                                                                                                                     |  📝    |

* 💚: Implemented and thoroughly tested.
* 💛: Implemented, but not thoroughly tested yet.
* ⏳: Implementation in progress.
* 📝: Implementation planned. Contributions welcome.

If a database is not on the list, we'd still love to support it. Open an issue
to discuss it.

Notes: 
- Because URLs allow many special characters, and may collide with the syntax of your command-line,
it's recommended to surround them with quotes. Alternatively, you may provide them in a TOML file via the `--config` option.
- For some databases, the arguments that you enter in the command line
may be case-sensitive. This is the case for the Snowflake schema and table names.