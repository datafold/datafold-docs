---
title: How to install
---

Installing Open Source Data Diff is as easy as using `pip`:

```
pip install data-diff
```

**Note:**

If you don't have the `pip` command, make sure you have Python 3.7+ installed, and that it is available in your path.

Throughout this guide, the command `pip` is used. Depending on your computer's setup, you MAY need to use `pip3` in place of `pip`. Once you've installed Python 3.7+, it's most likely that `pip` and `pip3` can be used interchangeably. Another equivalent option is to run `python -m pip`. Please reach out [via slack](https://locallyoptimistic.slack.com/archives/C03HUNGQV0S) if you're stuck.

## Install drivers

Then, you will need to install one or more driver(s) specific to the database(s) you want to connect to.*

- `pip install 'data-diff[mysql]'`

- `pip install 'data-diff[postgresql]'`

- `pip install 'data-diff[snowflake]'`

- `pip install 'data-diff[oracle]'`

- `pip install 'data-diff[presto]'`

- `pip install 'data-diff[trino]'`

- `pip install 'data-diff[clickhouse]'`

- `pip install 'data-diff[vertica]'`

- `pip install 'data-diff[duckdb]'`

- For BigQuery, see: https://pypi.org/project/google-cloud-bigquery/

- For redshift, install the postgresql driver.

You can also install several drivers at once:

```pip install 'data-diff[mysql,postgresql,snowflake]'```

_<sup>*</sup> Some drivers have dependencies that cannot be installed using `pip` and still need to be installed manually._