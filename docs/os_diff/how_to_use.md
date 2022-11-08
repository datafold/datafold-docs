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


#### Code Example: Diff Tables Between Databases
Here's an example command for your copy/pasting, taken from the screenshot above when we diffed data between Snowflake and Postgres.

```
data-diff \
  postgresql://<username>:'<password>'@localhost:5432/<database> \
  <table> \
  "snowflake://<username>:<password>@<password>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>" \
  <TABLE> \
  -k activity_id \
  -c activity \
  -w "event_timestamp < '2022-10-10'"
```

#### Code Example: Diff Tables Within a Database (available in pre release)

Here's a code example from [the video](about), where we compare data between two Snowflake tables within one database.

```
data-diff \
  "snowflake://<username>:<password>@<password>/<DATABASE>/<SCHEMA_1>?warehouse=<WAREHOUSE>&role=<ROLE>" <TABLE_1> \
  <SCHEMA_2>.<TABLE_2> \
  -k org_id \
  -c created_at -c is_internal \
  -w "org_id != 1 and org_id < 2000" \
  -m test_results_%t \
  --materialize-all-rows \
  --table-write-limit 10000
```

In both code examples, I've used `<>` carrots to represent values that **should be replaced with your values** in the database connection strings. For the flags (`-k`, `-c`, etc.), I opted for "real" values (`org_id`, `is_internal`) to give you a more realistic view of what your command will look like.

#### Databases we support and how to connect
| Database      | `DB_URI` string                                                                                                                   | Status |
|---------------|-------------------------------------------------------------------------------------------------------------------------------------|--------|
| PostgreSQL >=10    | `postgresql://<username>:'<password>'@<host>:5432/<database>`                                                                             |  💚    |
| MySQL         | `mysql://<username>:<password>@<hostname>:5432/<database>`                                                                              |  💚    |
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

Note: Because URLs allow many special characters, and may collide with the syntax of your command-line,
it's recommended to surround them with quotes. Alternatively, you may [provide them in a TOML file](#how-to-use-with-a-configuration-file) via the `--config` option.

#### Options

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

  **The following two options are not available when using the pre release In-DB feature:**

  - `--bisection-threshold` - Minimal size of segment to be split. Smaller segments will be downloaded and compared locally.
  - `--bisection-factor` - Segments per iteration. When set to 2, it performs binary search.

**In-DB commands, available in pre release only:**
  - `-m`, `--materialize` - Materialize the diff results into a new table in the database.
                            If a table exists by that name, it will be replaced.
                            Use `%t` in the name to place a timestamp.
                            Example: `-m test_mat_%t`
  - `--assume-unique-key` - Skip validating the uniqueness of the key column during joindiff, which is costly in non-cloud dbs.
  - `--sample-exclusive-rows` - Sample several rows that only appear in one of the tables, but not the other. Use with `-s`.
  - `--materialize-all-rows` -  Materialize every row, even if they are the same, instead of just the differing rows.
  - `--table-write-limit` - Maximum number of rows to write when creating materialized or sample tables, per thread. Default=1000.
  - `-a`, `--algorithm` `[auto|joindiff|hashdiff]` - Force algorithm choice



## How to use with a TOML configuration file

Data-diff lets you load the configuration for a run from [a TOML configuration file](https://toml.io/en/).

**Why use a TOML configuration file?**

- Convenience: Set up the parameters for diffs that need to run often.
- Easier and more readable: you can define the database connection settings as config values in a separate file, instead of a URI typed into the command line.
- Configurable: Gives you fine-grained control over the settings switches, without requiring any Python code.

In other words, you don't have to repeatedly enter verbose information such as URI strings or options every time you run a diff. This becomes even more handy as you use data-diff frequently in your workflow.

**How is a TOML configuration file set up for data-diff?**

There are two main sections:

- Database configuration
  - You can define one or more databases that will be used by runs.
- Run configuration
  - This is where you define default options that are inherited or overridden by specific runs.

By setting up a TOML file like this:

```
# DATABASE CONNECTION INFORMATION
# In this section, you can specify one or more databases that your runs (which are defined later in the file) can connect to.

# Specify the database connection information
[database.your_business_data] 
# Note that this line *can* be the same as your actual database name, but doesn't 
# have to be. It's a variable that stores information about a database connection 
# (driver, database, user, and password).

  driver = "postgresql"
  database = "your_business_data" # Note that this line *is* where your actual database name must be used.
  user = "your_username"
  password = "your_password"

# DATA-DIFF RUN PARAMETERS
# In this section, you can specify default run parameters that will be used by data-diff,
# as well as parameters for one or more named runs that you can use from the command line.

# Specify the default run parameters
[run.default]
  verbose = true

# Specify the run parameters for a run called 'analytics'
[run.analytics]

  # Source 1 ("left")
  1.database = "your_business_data" # This line instructs the 'analytics' run to 
  # use the database connection information defined in 'database.your_business_data' above.
  1.table = "orders" # This is a table in the 'your_business_data' database.

  # Source 2 ("right")
  2.database = "your_business_data" # Again, database.your_business_data is referenced.
  2.table = "orders_backup" # This is another table in the your_business_data database.

  verbose = false
```

Your command line input can look more like this:

```
data-diff \
  --conf ~/config_files/datadiff.toml \
  --run analytics \
  -k activity_id \
  -w "event_timestamp < '2022-10-10'"
```

- `--conf` specifies a path to the configuration file.
- `--run` specifies the parameters defined in your TOML file that will be used by data-diff.
- Optional: `-k` and `-w` are examples of parameters that have not been set in the TOML configuration file, and are defined in the command line.

In summary, the command above will compare between `orders` and `orders_backup` using the `analytics` run, while also using additional parameters: `-k` and `-w`.

**Note:** When defining how a run connects to a database, you can use a URI string (similar to the CLI command you would write in the absence of a TOML configuration file) instead of a database defined in `DATA-DIFF RUN PARAMETERS`:

```
  # Source 1 ("left")
  1.database = "postgresql://your_business_data:your_password/"
  1.table = "orders"
```

**Inheritance and overriding parameters**

If you review the TOML configuration file above, you'll see that that `verbose = true` is set the `run.default` parameters, which is then overridden by the `analytics` run to set `verbose = false`. 

If you decide you want to set `verbose = true` for a single run, you can switch that in your command line like this, by adding `-v` to override the `analytics` parameter:

```
data-diff \
  --conf ~/config_files/datadiff.toml \
  --run analytics \
  -k activity_id \
  -w "event_timestamp < '2022-10-10'" \
  -v
```

CLI switches have the final say, and will override the settings defined by the configuration file, and the current run.

**Where should you store your TOML configuration file?**

There are options!

- You could create a single file that has various configurations that are used for multiple projects. This might be stored in your home directory. This has the benefit of everything being in one place.
- Alternatively, you can have project-specific TOML files stored in each project. This has the benefit of each file being simpler and specific to a project. 
  - _Note: be sure to exclude the TOML file in your `.gitignore`, especially if it includes sensitive information such as passwords._

## How to use from Python

API reference: [https://data-diff.readthedocs.io/en/latest/](https://data-diff.readthedocs.io/en/latest/)

Example 1: Diff tables in mysql and postgresql

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

Example 2: Connect to snowflake using dictionary configuration

```python
SNOWFLAKE_CONN_INFO = {
    "driver": "snowflake",
    "user": "erez",
    "account": "whatever",
    "database": "TESTS",
    "warehouse": "COMPUTE_WH",
    "role": "ACCOUNTADMIN",
    "schema": "PUBLIC",
    "key": "snowflake_rsa_key.p8",
}

table1 = connect_to_table(SNOWFLAKE_CONN_INFO, "table_name")  # Uses id by default
```

Run `help(diff_tables)` or [read the docs](https://data-diff.readthedocs.io/en/latest/) to learn about the different options.




