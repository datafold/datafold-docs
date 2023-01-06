---
sidebar_position: 4
title: Options
---


You can use the following options to specify the configuration of a Open Source Data Diff run.

- **Config Key**: Use this syntax when using a [TOML configuration file](./how_to_use_with_toml.md) or [using Open Source Data Diff with Python](./how_to_use_with_python.md).
```
# Specify the default run parameters
[run.default]
  verbose = true
  stats = true
```

- **CLI Switch**: Use this syntax when writing a [command line input](./how_to_use_with_command_line.md).
```shell
data-diff DB1_URI TABLE1_NAME DB2_URI TABLE2_NAME --debug --v -k order_id
```

| Category | Config key | CLI&nbsp;switch&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;| Description | Within-Database | Cross-Database |
|---|---|---|---|---|---|
| | `help` | `--help` | Show help message and exit. | ✅ | ✅ |
| Schema | `key_columns`  | `-k` or `--key-columns` | Name of the primary key column. If none provided, default is 'id'. | ✅ | ✅ |
| Schema | `update_column` | `-t` or `--update-column` | Name of updated_at/last_updated column. | ✅ | ✅ |
| Schema | `columns` | `-c` or `--columns` | Names of extra columns to compare.  Can be used more than once in the same command. Accepts a name or a pattern, like in SQL. Example: `-c col% -c another_col -c %foorb.r%` | ✅ | ✅ |
| Schema | `assume_unique_key` | `--assume-unique-key` | Skip validating the uniqueness of the key column during joindiff, which is costly in non-cloud dbs. | ✅ |  |
| Filtering | `min_age` | `--min-age` | Considers only rows older than specified. Useful for specifying replication lag. Example: `--min-age=5min` ignores rows from the last 5 minutes. Valid units: `d, days, h, hours, min, minutes, mon, months, s, seconds, w, weeks, y, years` | ✅ | ✅ |
| Filtering | `max_age` | `--max-age` | Considers only rows younger than specified. See `--min-age`. | ✅ | ✅ |
| Filtering | `where` | `-w`, `--where` | An additional 'where' expression to restrict the search space. | ✅ | ✅ |
| Performance | `limit` | `-l` or `--limit` | Maximum number of differences to find (limits maximum bandwidth and runtime). | ✅ | ✅ |
| Performance | `threads` | `-j` or `--threads` | Number of worker threads to use per database. Default=1. | ✅ | ✅ |
| Performance | `algorithm`   | `-a`, `--algorithm` | Force algorithm choice. Options: `auto`, `joindiff`, `hashdiff` | ✅ | ✅ |
| Performance | `bisection_threshold` | `--bisection-threshold` | Minimal size of segment to be split. Smaller segments will be downloaded and compared locally. |   | ✅ |
| Performance | `bisection_factor` | `--bisection-factor` | Segments per iteration. When set to 2, it performs binary search. |   | ✅ |
| Output | `stats` | `-s` or `--stats` | Print stats instead of a detailed diff. | ✅ | ✅ |
| Output | `debug` | `-d` or `--debug` | Print debug info. | ✅ | ✅ |
| Output | `interactive` | `-i` or `--interactive` | Confirm queries, implies `--debug` | ✅ | ✅ |
| Output | `verbose` | `-v` or `--verbose` | Print extra info. | ✅ | ✅ |
| Output | `json` | `--json` | Print JSONL output for machine readability. | ✅ | ✅ |
| Output | `sample_exclusive_rows` | `--sample-exclusive-rows` | Sample several rows that only appear in one of the tables, but not the other. Use with `-s`. | ✅ |  |
| Output | `materialize_all_rows` | `--materialize-all-rows` | Materialize every row, even if they are the same, instead of just the differing rows. | ✅ |  |
| Output | `materialize` | `-m`, `--materialize` | Materialize the diff results into a new table in the database. If a table exists by that name, it will be replaced. Use `%t` in the name to place a timestamp. Example: `-m test_mat_%t` | ✅ |  |
| Output | `table_write_limit` | `--table-write-limit` | Maximum number of rows to write when creating materialized or sample tables, per thread. Default=1000. | ✅ |  |
| Settings | | `--conf`, `--run` | Specify the run and configuration [from a TOML file](how_to_use_with_toml.md). | ✅ | ✅ |
| Settings | `no_tracking` | `--no-tracking` | Open Source Data Diff sends home anonymous usage data. Use this to disable it. | ✅ | ✅ |
