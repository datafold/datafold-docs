---
sidebar_position: 2
title: Catalog + Column-level Lineage
---
## Catalog
![](../../static/img/catalog_landing.png)
With the Datafold Catalog, you can quickly see row counts, nulls, and column distributions for tables in your data warehouse.

To include more details in your Catalog, you can sync metadata from dbt and your data warehouse to create a single source of truth. You can implement custom tags to easily categorize and filter the data sets. 

## Column-level Lineage

![](../../static/img/lineage_detail.png)
After connecting your data warehouse, Datafold analyzes every SQL statement that is used to create the tables and views. Then, Datafold produces a graph of dependencies to easily visualize dependencies. See how data is produced and consumed - even correlated subqueries, `CASE WHEN` statements, and other complex queries are covered.

No additional developer resources are needed to unlock this capability. Simply connect your data warehouse and explore your lineage graph. 

To view the column-level Lineage of any table in your warehouse:

* Navigate to Catalog.
* Click on any table in the Catalog.
* Click on Lineage.