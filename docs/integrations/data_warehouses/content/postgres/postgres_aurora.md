---
sidebar_position: 1
title: PostgreSQL Aurora
---
# AWS Aurora Postgres Column-level Lineage

This will guide you through setting up Column-level Lineage with AWS Aurora using CloudWatch. AWS Aurora Serverless is not tied to any (virtual) machine, and therefore it only allows you to send the logs to CloudWatch.

**Steps to complete:**

1. [Setup Postgres with Permissions](postgres.md#run-sql-script)
2. [Increase the logging verbosity of Postgres](postgres_aurora.md#increased-verbosity) so Datafold can parse lineage
3. [Set up an account for fetching the logs from CloudWatch.](postgres_aurora.md#connect-datafold-to-cloudwatch)
4. [Configure your data source in Datafold](postgres_aurora.md#configure-in-datafold)

### Run SQL Script
To connect to Postgres, you need to create a user with read-only access to all tables in all schemas, write access to Datafold-specific schema for temporary tables:

```sql
/* Datafold utilizes a temporary dataset to materialize scratch work and keep data processing in the your warehouse. */

CREATE SCHEMA datafold_tmp;

/* Create a datafold user */

CREATE ROLE datafold WITH LOGIN ENCRYPTED PASSWORD 'SOMESECUREPASSWORD';

/* Give the datafole role write access to the temporary schema */

GRANT ALL ON SCHEMA datafold_tmp TO datafold;

/* Make sure that the postgres user has read permissions on the tables */

GRANT USAGE ON SCHEMA <myschema> TO datafold;
GRANT SELECT ON ALL TABLES IN SCHEMA <myschema> TO datafold;

```

### Increase logging verbosity

To begin, navigate to your database instances in the Amazon RDS sidebar.

![](<../../../../../static/img/psql_aurora_dbs.png>)

Then, create a new `Parameter Group`. Database instances run with default parameters that do not include logging verbosity. To turn on the logging verbosity, you'll need to create a new Parameter Group. Hit **Parameter Groups** on the menu and create a new Parameter Group.

![](<../../../../../static/img/psql_aurora_parameter_group.png>)

Next, select the `aurora-postgresql10` parameter group family. This depends on the cluster that you're running. For Aurora serverless, this is the appropriate family.

Finally, set the `log_statement` enum field to mod - meaning that it will log all the DDL statements, plus data-modifying statements. Note: This field isn't set by default. 

![](<../../../../../static/img/psql_aurora_logstatement.png>)

After saving the parameter group, go back to your database, and select the database cluster parameter group.

![](<../../../../../static/img/psql_aurora_clustergroup.png>)

### Connect Datafold to CloudWatch

Start by creating a new user to isolate the permissions as much as possible. Go to IAM and create a new user.

![](<../../../../../static/img/psql_aurora_iam_user.png>)

Next, create a new group named `CloudWatchLogsReadOnly` and attach the `CloudWatchLogsReadOnlyAccess` policy to it. Next, select the group.

![](<../../../../../static/img/psql_aurora_user_permissions.png>)


When reviewing the user, it should have the freshly created group attached to it.

![](<../../../../../static/img/psql_aurora_user_review.png>)

After confirming the new user you should be given the `Access Key` and `Secret Key`. Save these two codes securely to finish configurations on Datafold. 

The last piece of information Datafold needs is the CloudWatch Log Group. You will find this in CloudWatch under the Log Group section in the sidebar. It will be formatted as `/aws/rds/cluster/<my_cluster_name>/postgresql`.

![](<../../../../../static/img/psql_aurora_log_group.png>)

### Configure in Datafold

| Field Name      | Description |
| ----------- | ----------- |
| Name     | A name given to the data source within Datafold |
| Host   | The hostname address for your database; default value `127.0.0.1` |
| Port   | Postgres connection port; default value is 5432 |
| User   | The user role created in the SQL script; `datafold`  |
| Password  | The password created in the SQL permissions script |
| Database Name  | The name of the Postgres database you want to connect to |
| AWS Access Key  | The Access Key provided in the [Connect Datafold to CloudWatch](postgres_aurora.md#connect-datafold-to-cloudwatch) step|
| AWS Secret  | The Secret Key provided in the [Connect Datafold to CloudWatch](postgres_aurora.md#connect-datafold-to-cloudwatch) step |
| Cloudwatch Postgres Log Group  | The path of the Log Group; formatted as `/aws/rds/cluster/<my_cluster_name>/postgresql` |
| Schema for temporary tables  | The schema created in the SQL setup script; `datafold_tmp` |

Click **Create**. Your data source is ready!