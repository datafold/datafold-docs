---
sidebar_position: 4
title: Examples
---

# Examples

## Command Line Examples
For each of the 💚 implemented and thoroughly tested 💚 databases, we'll provide examples of `data-diff` input code and outputs on various data structures and sizes. 

### Snowflake

#### Massive Data Set with Missing Row

[TODO Example to be added]

#### Smaller Data Set with Several Conflicting Values

[TODO add a few screenshots to break up this text]

In this example, we use the CLI to compare a small data set of organizations (in this case, pharmacies and groceries) that exists as the `COMPANIES` table in two schemas, `ANALYTICS` AND `ANALYTICS_DEV`, in the same database, `SNOWFLAKE_DB`. 

Recall the basic command line formula mentioned in TODO link to how to use

`data-diff DB1_URI TABLE1_NAME DB2_URI TABLE2_NAME [OPTIONS]`

We'll used `-k` to specify that the primary key is `org_id`, and `-c` to indicate that we'd also like to surface conflicts in the`amount` and `company_name` columns.

Note that `ANALYTICS` is the first data set in your command (`DB1_URI`), and `ANALYTICS_DEV` is the second data set in your command (`DB2_URI`). This will be important to remember when interpreting the results.

All other strings surrounded in `<>` carrots 🥕 should be replaced with your information.

Looking at the command and the results below, we find:

- Walgreens, CVS, DuaneReade, and Albertsons are missing from `ANALYTICS_DEV` (the `-` indicates that the row is _missing_ from `DB2_URI`)
- Aldi is missing from `ANALYTICS` (the `+` indicates that it _appeared_ in `DB2_URI`)
- RiteAid is in both data sets, which is why it appears twice. But something else is wrong: the `amount` changed!
- All rows that match exactly are not printed out.

```
$ data-diff \
  "snowflake://YOUR_USERNAME:<your_password>@<YOUR_ACCOUNT>/SNOWFLAKE_DB/ANALYTICS?warehouse=<YOUR_WAREHOUSE>&role=<YOUR_ROLE>" COMPANIES \
  "snowflake://YOUR_USERNAME:<your_password>@<YOUR_ACCOUNT>/SNOWFLAKE_DB/ANALYTICS_DEV?warehouse=<YOUR_WAREHOUSE>&role=<YOUR_ROLE>" COMPANIES " COMPANIES \
  -k org_id \
  -c company_name -c amount

- 1, 5000.00, Walgreens
- 2, 9000.00, CVS
+ 3, 49900.00 Aldi
- 4, 710000.00, DuaneReade
- 5, 21000.00, RiteAid
+ 5, 2000.00, RiteAid
- 6, 0.00, Albertsons
```

### PostgreSQL

#### Massive Data Set with Missing Row

In this example, we use the CLI to compare 25M rows in PostgreSQL where the
right-hand table is missing single row with `id=12500048`:

```
$ data-diff \
    postgresql://user:password@localhost:5432/database rating \
    postgresql://user:password@localhost:5432/database rating_del1 \
    --bisection-threshold 100000 \ # for readability, try default first
    --bisection-factor 6 \ # for readability, try default first
    --update-column timestamp \
    --verbose

    # Consider running with --interactive the first time.
    # Runs `EXPLAIN` for you to verify the queries are using indexes.
    # --interactive
[10:15:00] INFO - Diffing tables | segments: 6, bisection threshold: 100000.
[10:15:00] INFO - . Diffing segment 1/6, key-range: 1..4166683, size: 4166682
[10:15:03] INFO - . Diffing segment 2/6, key-range: 4166683..8333365, size: 4166682
[10:15:06] INFO - . Diffing segment 3/6, key-range: 8333365..12500047, size: 4166682
[10:15:09] INFO - . Diffing segment 4/6, key-range: 12500047..16666729, size: 4166682
[10:15:12] INFO - . . Diffing segment 1/6, key-range: 12500047..13194494, size: 694447
[10:15:13] INFO - . . . Diffing segment 1/6, key-range: 12500047..12615788, size: 115741
[10:15:13] INFO - . . . . Diffing segment 1/6, key-range: 12500047..12519337, size: 19290
[10:15:13] INFO - . . . . Diff found 1 different rows.
[10:15:13] INFO - . . . . Diffing segment 2/6, key-range: 12519337..12538627, size: 19290
[10:15:13] INFO - . . . . Diffing segment 3/6, key-range: 12538627..12557917, size: 19290
[10:15:13] INFO - . . . . Diffing segment 4/6, key-range: 12557917..12577207, size: 19290
[10:15:13] INFO - . . . . Diffing segment 5/6, key-range: 12577207..12596497, size: 19290
[10:15:13] INFO - . . . . Diffing segment 6/6, key-range: 12596497..12615788, size: 19291
[10:15:13] INFO - . . . Diffing segment 2/6, key-range: 12615788..12731529, size: 115741
[10:15:13] INFO - . . . Diffing segment 3/6, key-range: 12731529..12847270, size: 115741
[10:15:13] INFO - . . . Diffing segment 4/6, key-range: 12847270..12963011, size: 115741
[10:15:14] INFO - . . . Diffing segment 5/6, key-range: 12963011..13078752, size: 115741
[10:15:14] INFO - . . . Diffing segment 6/6, key-range: 13078752..13194494, size: 115742
[10:15:14] INFO - . . Diffing segment 2/6, key-range: 13194494..13888941, size: 694447
[10:15:14] INFO - . . Diffing segment 3/6, key-range: 13888941..14583388, size: 694447
[10:15:15] INFO - . . Diffing segment 4/6, key-range: 14583388..15277835, size: 694447
[10:15:15] INFO - . . Diffing segment 5/6, key-range: 15277835..15972282, size: 694447
[10:15:15] INFO - . . Diffing segment 6/6, key-range: 15972282..16666729, size: 694447
+ (12500048, 1268104625)
[10:15:16] INFO - . Diffing segment 5/6, key-range: 16666729..20833411, size: 4166682
[10:15:19] INFO - . Diffing segment 6/6, key-range: 20833411..25000096, size: 4166685
```

#### Smaller Data Set with Conflicting Values

In this example, we use the CLI to compare a smaller data set with one missing row (`where actor_id = 4`) as well as several conflicting values:
- `Ed` is misspelled as `Edd`. (Or is `Edd` misspelled as `Ed`? Hard to say!)
- `Ed`/`Edd` and `Nick` have conflicting `last_update` values.
- There's one more conlicting `first_name` spelling:  `Penelope` vs `Penelop`.

```
 $ data-diff \
  postgresql://user:password@localhost/database actor \
  postgresql://user:password@localhost/database actor2 \
  -k actor_id \
  -c first_name -c last_name -c last_update
- 1, 2013-05-26 14:47:57.620000, Penelope, Guiness
+ 1, 2013-05-26 14:47:57.620000, Penelop, Guiness
- 2, 2013-05-26 14:47:57.620000, Nick, Wahlberg
+ 2, 2022-10-10 13:36:12.435821, Nick, Wahlberg
- 3, 2013-05-26 14:47:57.620000, Ed, Chase
+ 3, 2022-10-10 13:31:28.214738, Edd, Chase
- 4, 2013-05-26 14:47:57.620000, Jennifer, Davis
```

### MySQL

#### Massive Data Set with Missing Row

[TODO Example to be added]

#### Smaller Data Set with Conflicting Values

[TODO Example to be added]

### BigQuery

#### Massive Data Set with Missing Row

[TODO Example to be added]

#### Smaller Data Set with Conflicting Values

[TODO Example to be added]

### Redshift

#### Massive Data Set with Missing Row

[TODO Example to be added]

#### Smaller Data Set with Conflicting Values

[TODO Example to be added]



## Python Examples

API reference: [https://data-diff.readthedocs.io/en/latest/](https://data-diff.readthedocs.io/en/latest/)

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