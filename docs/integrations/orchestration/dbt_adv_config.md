---
sidebar_position: 3
title: dbt - Adv. configs
description: Advanced configuration options for dbt projects
---

# dbt - Advanced Configurations

### Prerequisites

- To configure dbt Cloud, you must first connect a [Data Source](integrations/data_warehouses/dw_overview.md) and connect a [GitHub](/integrations/git/github.md) or [GitLab](/integrations/git/gitlab.md) account.
## Primary Keys
### **Tag primary keys in dbt models**

Datafold needs to know which column is the primary key of the table to perform the diff. When Datafold cannot determine the primary key of the two tables to diff, it will produce an error:

![](../../../static/img/primary_key_not_set.png)

When setting up the CI integration, one of the steps is providing the primary key tag:

![](../../../static/img/primary_key_tag.png)

We use this in the dbt metadata to let Datafold know which column can be used to perform the diff. Datafold supports composite primary keys, meaning that you can assign multiple columns that make up the primary key together.

### Metadata

The first option is setting the `primary-key` key in the dbt metadata. There are [several ways to configure this](https://docs.getdbt.com/reference/resource-configs/meta) in your dbt project using either the `meta` key in a yaml file or a model-specific config block.

Here's an example of how you would specify that the `user_id` column is the primary key of the `users` table in a yaml file.

```yaml
models:
  - name: users
    columns:
      - name: user_id
        meta:
          primary-key: true
```

In the case of a compound Primary Key, you can assign a second column (or more):
```yaml
models:
  - name: users
    columns:
      - name: user_id
        meta:
          primary-key: true
      - name: version_id
        meta:
          primary-key: true
```

### Tags

If the primary key is not found in the metadata, it will go through the [tags](https://docs.getdbt.com/reference/resource-properties/tags).

```yaml
models:
  - name: users
    columns:
      - name: user_id
        tags:
          - primary-key
```
In the case of a compound Primary Key, you can assign a second column (or more):
```yaml
models:
  - name: users
    columns:
      - name: version_id
        tags:
          - primary-key
```

### Inferred

If the primary key isn't provided explicitly, Datafold will try to assume a primary key from dbt's uniqueness tests. If you have a single column uniqueness test defined, it will use this column as the PK.

```yaml
models:
  - name: users
    columns:
      - name: user_id
        tests:
          - unique
```

Also, model level uniqueness tests are used for inferring the PK.

```yaml
models:
  - name: sales
    columns:
      - name: col1
      - name: col2
      ...
    tests:
      - unique:
          column_name: "col1 || col2"

  - name: sales
    columns:
      - name: col1
      - name: col2
      ...
    tests:
      - unique:
          column_name: "CONCAT(col1, col2)"
```
### Combination of Columns Primary Key
Finally, we support `unique_combination_of_columns` from the `dbt_utils` package:

```yaml
models:
  - name: users
    columns:
      - name: order_no
      - name: order_line
      ...
    tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
            - order_no
            - order_line
```

Keep in mind that this is a failover mechanism. If you change the uniqueness test, this will also impact the way Datafold performs the diff.

### Checking primary key annotations

You can check what models in your dbt repo already have primary key annotations, and which need more attention. You'll need to install Datafold SDK and configure access parameters.

```bash
$ pip3 install 'datafold-sdk'

# skip this step if you are using app.datafold.com
$ export DATAFOLD_HOST=https://<hostname>

# get your API key in Datafold UI -> Edit Profile -> API Key
$ export DATAFOLD_APIKEY=Ratc0zd27hjkhjfgPU3AxZ
```

After that, you need to compile `manifest.json`, and you'll be ready to do the check.

```bash
# Lookup your CI configuration id in URL when you go to Settings -> CI settings -> <name>:
# https://app.datafold.com/settings/ci_integrations/14

$ datafold dbt check-primary-keys --ci-config-id 14 manifest.json 
meta        dbt_snowflake.service_calls              INCIDENT_NUMBER            models/service_calls.sql              models/schema.yml 
meta        dbt_snowflake.supply_of_ones             ID                         models/supply_of_ones.sql             models/schema.yml 
none        dbt_snowflake.fokko.boom                                            models/fokko/boom.sql                                   
none        dbt_snowflake.new_service_calls                                     models/new_service_calls.sql          models/schema.yml 
tags        dbt_snowflake.ephemeral_supply_of_twos   ID                         models/ephemeral_supply_of_twos.sql   models/schema.yml 
uniqueness  dbt_snowflake.new_service_calls_concat2  CAL_YEAR, INCIDENT_NUMBER  models/new_service_calls_concat2.sql  models/schema.yml 
uniqueness  dbt_snowflake.supply_of_twos             ID                         models/supply_of_twos.sql             models/schema.yml 
```

The first column shows how the key was inferred:

* `none` - Datafold was unable to find any PKs,
* `uniqueness` - primary keys were derived from uniqueness tests,
* `tags` - PKs were specified with column-level tags,
* `meta` - column-level metadata was used,
* `meta_table` - table-level metadata.

Out of those, `none` and possibly `uniqueness` require further actions.

The other fields in the printout are:

* fully qualified name of dbt model,
* list of primary keys,
* sql file that contains model definition,
* "patch" yml file that has dbt configuration of the model.


## Other Advanced Configuration
Table metadata can also be used to specify per-model diff options. 

### Filter
In the example below, diff is configured to compare only rows matching `user_id > 2350`. The expression in the filter is a SQL expression, and can be anything you could put into `where` clause when selecting from the tables.

```yaml
models:
  - name: users
    meta:
      datafold:
        datadiff:
          filter: "user_id > 2350"
```

This field is also commonly used to scope diffs to a small time period.

Example: `filter: "source_timestamp >= current_date() - 7"`

### Include and Exclude
You can include (`include_columns`) or exclude (`exclude_columns`) columns in the diff.

```yaml
models:
  - name: users
    meta:
      datafold:
        datadiff:
          include_columns:
            - user_id
            - created_at
            - name
          exclude_columns:
            - full_name
```
### Time Travel
For time dimensions, you can set a time column of \`time\_travel\` field. If you want to add time travelling, you can define `prod_time_travel` for travelling of a production model,  `pr_time_travel` of a PR model. Please keep in mind that not all databases support time traveling.                   

```yaml
models:
  - name: users
    meta:
      datafold:
        datadiff:
          prod_time_travel:
            - 2022-02-07T00:00:00
          pr_time_travel:
            - 2022-02-07T00:00:00
          time_column:
            - created_at
```

### dbt metadata synchronization

Datafold integrates very well with dbt, and also has the ability to ingest the metadata provided by dbt automatically. dbt models have metadata that can be synchronized from the production branch into the Datafold catalog. When a table has metadata being synchronized using dbt, user editing is no longer permitted for that entire table. This is to ensure that there is a single source of truth.

Metadata can be applied both on a table and column level.

```yaml
models:
  - name: users
    description: "Description of the table"
    meta:
      owner: user@company.com
      foo: bar
    tags:
      - pii
      - abc
    columns:
      - name: user_id
        tags:
          - pk
          - id
        meta:
          pk: true
      - name: email
        description: "The user's email"
        tags:
          - pii
        meta:
          type: email
```

There are two special meta types:

* `owner`: Used to specify the owner of the table and applies the owner of the table in the catalog view.
* `<pk_tag>`: The tag/name that is configured to identify primary columns is not synchronized into the meta-information, but it is synchronized as a tag if it exists.

So for the above table:

* `description` is synchronized into the description field of the table in the catalog.
* The `owner` of the table is set to the user identified by the `user@company.com` field. This user must exist in Datafold with that email.
* The `foo` meta information is added to the description field with the value `bar`.
* The tags `pii` and `bar` are applied to the table as tags.

For the columns above:

* The column `user_id` has two tags applied: `pk` and `id`.
* The metadata for `user_id` is ignored, because it reflects the primary key tag.
* The `email` column has the description applied.
* The `email` column has the tag `pii` applied.
* The `email` column has extra metadata information in the description field: `type` with the value `email`.

Metadata synchronization occurs in one of two methods:

* The `meta_schedule` is set for the dbt cloud integration. This will run according to the specified cron schedule, find the most recent dbt cloud production run, and synchronize the metadata from there.
* It can also be configured to synchronize metadata whenever a push to production happens.
