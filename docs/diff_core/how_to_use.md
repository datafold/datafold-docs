---
sidebar_position: 3
title: How to use
---

# How to use

## How to use from the command-line

To run `data-diff` from the command line, run this command:

`data-diff DB1_URI TABLE1_NAME DB2_URI TABLE2_NAME [OPTIONS]`

Let's break this down. Assume there are two tables stored in two databases, and you want to know the differences between those tables.

- `DB1_URI` will be a string that `data-diff` uses to connect to the database where the first table is stored.
- `TABLE1_NAME` is the name of the table in the `DB1_URI` database.
- `DB2_URI` will be a string that `data-diff` uses to connect to the database where the second table is stored.
- `TABLE2_NAME` is the name of the second table in the `DB2_URI` database.
- `[OPTIONS]` can be replaced with a variety of additional commands, [detailed here](#options).



| Database      | Connection string                                                                                                                   | Status |
|---------------|-------------------------------------------------------------------------------------------------------------------------------------|--------|
| PostgreSQL >=10    | `postgresql://'<user>':<password>@<host>:5432/<database>`                                                                             |  💚    |
| MySQL         | `mysql://<user>:<password>@<hostname>:5432/<database>`                                                                              |  💚    |
| Snowflake     | **With password:**`"snowflake://<USER>:<password>@<ACCOUNT>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>"`<br />**With SSO:** `"snowflake://<USER>@<ACCOUNT>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>&authenticator=externalbrowser"`<br />_Note: Unless something is explicitly case sensitive (like your password) use all caps._ |  💚    |
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

### Options:

  - `--help` - Show help message and exit.
  - `-k` or `--key-columns` - Name of the primary key column. If none provided, default is 'id'.
  - `-t` or `--update-column` - Name of updated_at/last_updated column
  - `-c` or `--columns` - Names of extra columns to compare.  Can be used more than once in the same command.
                          Accepts a name or a pattern like in SQL.
                          Example: `-c col% -c another_col -c %foorb.r%`
  - `-l` or `--limit` - Maximum number of differences to find (limits maximum bandwidth and runtime)
  - `-s` or `--stats` - Print stats instead of a detailed diff
  - `-d` or `--debug` - Print debug info
  - `-v` or `--verbose` - Print extra info
  - `-i` or `--interactive` - Confirm queries, implies `--debug`
  - `--json` - Print JSONL output for machine readability
  - `--min-age` - Considers only rows older than specified. Useful for specifying replication lag.
                  Example: `--min-age=5min` ignores rows from the last 5 minutes.
                  Valid units: `d, days, h, hours, min, minutes, mon, months, s, seconds, w, weeks, y, years`
  - `--max-age` - Considers only rows younger than specified. See `--min-age`.
  - `-j` or `--threads` - Number of worker threads to use per database. Default=1.
  - `-w`, `--where` - An additional 'where' expression to restrict the search space.
  - `--conf`, `--run` - Specify the run and configuration from a TOML file. (see below)
  - `--no-tracking` - data-diff sends home anonymous usage data. Use this to disable it.
  - `-a`, `--algorithm` `[auto|joindiff|hashdiff]` - Force algorithm choice

**Same-DB diff only:**
  - `-m`, `--materialize` - Materialize the diff results into a new table in the database.
                            If a table exists by that name, it will be replaced.
                            Use `%t` in the name to place a timestamp.
                            Example: `-m test_mat_%t`
  - `--assume-unique-key` - Skip validating the uniqueness of the key column during joindiff, which is costly in non-cloud dbs.
  - `--sample-exclusive-rows` - Sample several rows that only appear in one of the tables, but not the other. Use with `-s`.
  - `--materialize-all-rows` - Materialize every row, even if they are the same, instead of just the differing rows.
  - `--table-write-limit` - Maximum number of rows to write when creating materialized or sample tables, per thread. Default=1000.

**Cross-DB diff only:**
  - `--bisection-threshold` - Minimal size of segment to be split. Smaller segments will be downloaded and compared locally.
  - `--bisection-factor` - Segments per iteration. When set to 2, it performs binary search.



### How to use with a configuration file

Data-diff lets you load the configuration for a run from a TOML file.

**Reasons to use a configuration file:**

- Convenience - Set-up the parameters for diffs that need to run often

- Easier and more readable - you can define the database connection settings as config values, instead of in a URI.

- Gives you fine-grained control over the settings switches, without requiring any Python code.

Use `--conf` to specify that path to the configuration file. data-diff will load the settings from `run.default`, if it's defined.

Then you can, optionally, use `--run` to choose to load the settings of a specific run, and override the settings `run.default`. (all runs extend `run.default`, like inheritance).

Finally, CLI switches have the final say, and will override the settings defined by the configuration file, and the current run.

Example TOML file:

```toml
# Specify the connection params to the test database.
[database.test_postgresql]
driver = "postgresql"
user = "postgres"
password = "Password1"

# Specify the default run params
[run.default]
update_column = "timestamp"
verbose = true

# Specify params for a run 'test_diff'.
[run.test_diff]
verbose = false
# Source 1 ("left")
1.database = "test_postgresql"                      # Use options from database.test_postgresql
1.table = "rating"
# Source 2 ("right")
2.database = "postgresql://postgres:Password1@/"    # Use URI like in the CLI
2.table = "rating_del1"
```

In this example, running `data-diff --conf myconfig.toml --run test_diff` will compare between `rating` and `rating_del1`.
It will use the `timestamp` column as the update column, as specified in `run.default`. However, it won't be verbose, since that
flag is overwritten to `false`.

Running it with `data-diff --conf myconfig.toml --run test_diff -v` will set verbose back to `true`.

## How to use from Python

API reference: [https://data-diff.readthedocs.io/en/latest/](https://data-diff.readthedocs.io/en/latest/)

Example:

```python
# Optional: Set logging to display the progress of the diff
import logging
logging.basicConfig(level=logging.INFO)

from data_diff import connect_to_table, diff_tables

table1 = connect_to_table("postgresql:///", "table_name", "id")
table2 = connect_to_table("mysql:///", "table_name", "id")

for different_row in diff_tables(table1, table2):
    plus_or_minus, columns = different_row
    print(plus_or_minus, columns)
```

Run `help(diff_tables)` or [read the docs](https://data-diff.readthedocs.io/en/latest/) to learn about the different options.




