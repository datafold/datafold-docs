---
title: Databases we support
---

#### Databases we support and how to connect

If a database is not on the list, we'd still love to support it. Please [open an issue](https://github.com/datafold/data-diff/issues) to discuss it.

| Database      | `DB_URI` string                                                                                                                   | Status |
|---------------|-------------------------------------------------------------------------------------------------------------------------------------|--------|
| PostgreSQL >=10    | `postgresql://<username>:'<password>'@<host>:5432/<database>`                                                                             |  💚    |
| MySQL         | `mysql://<username>:<password>@<hostname>:5432/<database>`                                                                              |  💚    |
| Snowflake     | **With password:**`"snowflake://<USER>:<password>@<ACCOUNT>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>"`<br />**With SSO:** `"snowflake://<USER>@<ACCOUNT>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>&authenticator=externalbrowser"`<br />_Note: Unless something is explicitly case sensitive (like your password) use all caps._ |  💚    |
| BigQuery      | `bigquery://<project>/<dataset>`                                                                                                    |  💚    |
| Redshift      | `redshift://<username>:<password>@<hostname>:5439/<database>`                                                                       |  💚    |
| Oracle        | `oracle://<username>:<password>@<hostname>/database`                                                                                |  💛    |
| Presto        | `presto://<username>:<password>@<hostname>:8080/<database>`                                                                         |  💛    |
| Databricks    | `databricks://:<access_token>@<server_name>/<http_path>?catalog=<catalog>&schema=<schema>`                                          |  💛    |
| Trino         | `trino://<username>:<password>@<hostname>:8080/<database>`                                                                          |  💛    |
| Clickhouse    | `clickhouse://<username>:<password>@<hostname>:9000/<database>`                                                                     |  💛    |
| Vertica       | `vertica://<username>:<password>@<hostname>:5433/<database>`                                                                        |  💛    |
| DuckDB        | `duckdb://<database>@<dbpath>`                                                                                                      |  💛    |
| SQLite        |                                                                                                                                     |  📝    |

* 💚: Implemented and thoroughly tested.
* 💛: Implemented, but not thoroughly tested yet.
* ⏳: Implementation in progress.
* 📝: Implementation planned. Contributions welcome.

Note: Because URLs allow many special characters, and may collide with the syntax of your command-line,
it's recommended to surround them with quotes. Alternatively, you may [provide them in a TOML file](./how_to_use/how_to_use_with_toml).
