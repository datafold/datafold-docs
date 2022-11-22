---
title: How to install
---

First, install `data-diff` using `pip`.

```
pip install data-diff
```

To try out bleeding-edge features, including materialization of results in your data warehouse:

```
pip install data-diff --pre
```

**Note:** Throughout this guide, the command `pip` is used. Depending on your computer's setup, you MAY need to use `pip3` in place of `pip`. Once you've installed Python 3.7+, it's most likely that `pip` and `pip3` can be used interchangeably. Please reach out [via slack](https://locallyoptimistic.slack.com/archives/C03HUNGQV0S) if you're stuck.

## Install drivers

Then, you will need to install one or more driver(s) specific to the database(s) you want to connect to.*

- `pip install 'data-diff[mysql]'`

- `pip install 'data-diff[postgresql]'`

- `pip install 'data-diff[snowflake]'`

- `pip install 'data-diff[presto]'`

- `pip install 'data-diff[oracle]'`

- `pip install 'data-diff[trino]'`

- `pip install 'data-diff[clickhouse]'`

- `pip install 'data-diff[vertica]'`

- `pip install 'data-diff[duckdb]'`

- For BigQuery, see: https://pypi.org/project/google-cloud-bigquery/

You can also install several drivers at once:

```pip install 'data-diff[mysql,postgresql,snowflake]'```

_<sup>*</sup> Some drivers have dependencies that cannot be installed using `pip` and still need to be installed manually._