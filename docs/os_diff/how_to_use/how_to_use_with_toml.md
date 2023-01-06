---
sidebar_position: 2
title: TOML configuration file
---

Open Source Data Diff lets you load the configuration for a run from [a TOML configuration file](https://toml.io/en/).

**Why use a TOML configuration file?**

- Convenience: Set up the parameters for diffs that need to run often.
- Easier and more readable: you can define the database connection settings as config values in a separate file, instead of a URI typed into the command line.
- Configurable: Gives you fine-grained control over the settings switches, without requiring any Python code.

In other words, you don't have to repeatedly enter verbose information such as URI strings or options every time you run a diff. This becomes even more handy as you use Open Source Data Diff frequently in your workflow.

**How is a TOML configuration file set up for Open Source Data Diff?**

There are two main sections:

- Database configuration
  - You can define one or more databases that will be used by runs.
- Run configuration
  - This is where you define default options that are inherited or overridden by specific runs.

By setting up a TOML file like this:

```python
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

# Open Source Data Diff RUN PARAMETERS
# In this section, you can specify default run parameters that will be used by Open Source Data Diff,
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

```shell
data-diff \
  --conf ~/config_files/datadiff.toml \
  --run analytics \
  -k activity_id \
  -w "event_timestamp < '2022-10-10'"
```

- `--conf` specifies a path to the configuration file.
- `--run` specifies the parameters defined in your TOML file that will be used by Open Source Data Diff.
- Optional: `-k` and `-w` are examples of parameters that have not been set in the TOML configuration file, and are defined in the command line.

In summary, the command above will compare between `orders` and `orders_backup` using the `analytics` run, while also using additional parameters: `-k` and `-w`.

**Note:** When defining how a run connects to a database, you can use a URI string (similar to the CLI command you would write in the absence of a TOML configuration file) instead of a database defined in `OPEN SOURCE DATA DIFF RUN PARAMETERS`:

```python
  # Source 1 ("left")
  1.database = "postgresql://your_business_data:your_password/"
  1.table = "orders"
```

**Setting up a Snowflake database connection in your TOML file**

Because Snowflake has several unique settings that aren't used to connect to other databases, your TOML configuration needs to be structured a bit differently.

Here's an example of how to configure a Snowflake database in a TOML configuration file. In this example, the Open Source Data Diff run is configured to compare Postgres data with Snowflake data, as you would when validating cross-database replication.

```python
# DATABASE CONNECTION INFORMATION

# Specify the database connection information
[database.your_postgres_business_data] 

  driver = "postgresql"
  database = "your_business_data"
  user = "your_username"
  password = "your_password"

[database.your_snowflake_business_data]
  driver = "snowflake"
  database = "YOUR_BUSINESS_DATA"
  user = "YOUR_USERNAME"
  password = "YOUR_PASSWORD"
  account = "YOUR_SNOWFLAKE_ACCOUNT"
  schema = "YOUR_SCHEMA"
  warehouse = "YOUR_WAREHOUSE"
  role = "YOUR_ROLE"

### RUN PARAMS: xdb_validation
# Specify the run parameters for a run called xdb_validation
[run.xdb_validation]

  # Source 1 ("left")
  1.database = "your_postgres_business_data"
  1.table = "orders"

  # Source 2 ("right")
  2.database = "your_snowflake_business_data"
  2.table = "ORDERS"
```

Then, you can run the following in your command line, which will inherit the configurations.

```shell
data-diff \
  --conf ~/config_files/datadiff.toml \
  --run xdb_validation \
  -k activity_id \
  -c activity \
  -w "event_timestamp < '2022-10-10'"
```

**Inheritance and overriding parameters**

If you review the TOML configuration file above, you'll see that that `verbose = true` is set the `run.default` parameters, which is then overridden by the `analytics` run to set `verbose = false`. 

If you decide you want to set `verbose = true` for a single run, you can switch that in your command line like this, by adding `-v` to override the `analytics` parameter:

```shell
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
