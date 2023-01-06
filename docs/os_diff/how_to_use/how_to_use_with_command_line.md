---
sidebar_position: 1
title: Command line
---

Once you've installed Open Source Data Diff using `pip`, it should be available through the `data-diff` command.

Run it without any arguments to get the "help" text:

```shell
$ data-diff
data-diff v0.3.0 - efficiently diff rows across database tables.

Usage:
  * In-db diff:    python -m data_diff <database_a> <table_a> <table_b> [OPTIONS]
  * Cross-db diff: python -m data_diff <database_a> <table_a> <database_b> <table_b> [OPTIONS]
  * Using config:  python -m data_diff --conf PATH [--run NAME] [OPTIONS]

Options:
  ... (continues with the list of command-line switches) ...
```

"In-db" diffs two tables within the same database, while "cross-db" diffs across tables in different databases.

Let's break this down. Assume there are two tables stored in two different databases, and you want to know the differences between those tables.

- `database_a` will be a string that Open Source Data Diff uses to connect to the database where the first table is stored.
- `table_a` is the name of the table in the first database.
- `database_b` will be a string that Open Source Data Diff uses to connect to the database where the second table is stored.
- `table_b` is the name of the second table in the second database.
- `[OPTIONS]` can be replaced with a variety of additional commands, [detailed here](#options).

Usually, `database_a` and `database_b` will be URL connection strings. However, when `--conf` is specified, they can also be the names of the database configurations defined in a [TOML configuration file](./how_to_use_with_toml).

Note that if `database_a` and `database_b` are the same, Open Source Data Diff will automatically opt for using the in-db diff, despite the "cross-db syntax". To force a specific algorithm, you can use the `--algorithm` switch.


#### Code Example: Diff Tables Between Databases
Here's an example command for your copy/pasting, taken from the screenshot above when we diffed data between Snowflake and Postgres.

```shell
data-diff \
  postgresql://<username>:'<password>'@localhost:5432/<database> \
  <table> \
  "snowflake://<username>:<password>@<password>/<DATABASE>/<SCHEMA>?warehouse=<WAREHOUSE>&role=<ROLE>" \
  <TABLE> \
  -k activity_id \
  -c activity \
  -w "event_timestamp < '2022-10-10'"
```

What these switches mean:

- `-k` sets the key column. That is the column that is used for determining the identity of the row. For the best performance, it should be unique and uniformly distributed. The identity is used to distinguish whether the rows were added/removed or updated.
- `-c` is an additional, non-key column to diff.
- `-w`, or `--where`, is an arbitrary SQL expression to filter the table.

Be sure to review and reference all the options you may use to construct your Open Source Data Diff commands, which are documented [here](./options).

#### Code Example: Diff Tables Within a Database

Here's a code example corresponding to the (within-database demo video)[https://www.loom.com/share/682e4b7d74e84eb4824b983311f0a3b2], where we compare data between two Snowflake tables within one database.

```shell
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

`-m` materializes the results into the specified table. `%t` will get replaced by the current timestamp.

In both code examples, we've used `<>` carrots to represent values that **should be replaced with your values** in the database connection strings. For the flags (`-k`, `-c`, etc.), we opted for "real" values (`org_id`, `is_internal`) to give you a more realistic view of what your command will look like.

#### Getting tired of entering all this text into the command line?

Consider using a [TOML configuration file](./how_to_use_with_toml)!