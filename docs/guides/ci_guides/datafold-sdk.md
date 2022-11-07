---
sidebar_position: 3
title: datafold-sdk
---

Datafold allows you to trigger data diffs from CI using the [datafold-sdk](https://pypi.org/project/datafold-sdk/). This allows you to easily integrate Datafold in your CI with arbitrary pipeline orchestrators (e.g. Airflow, Dagster, Prefect).

## Prerequisites
** The key prerequisite is that Datafold has access to two datasets to compare. ** If your PR CI deploys changes to a staging location and materializes a staging version of the involved tables, Datafold can be added as a final step to diff dev and production.

With a custom data pipeline, Datafold just needs to know what objects to compare and the primary key column, for example:
- staging_database.sales.orders
- production_database.sales.orders
- pk: order_id

- Additional prerequisites:
    - Existing [Github](/docs/integrations/git/github.md) or [Gitlab](/docs/integrations/git/gitlab.md) integration
    - Existing [Data Warehouse](/docs/integrations/data_warehouses/dw_overview.md) integration

## Config

- ** Admin > Settings > Orchestration > + Add new integration **

![](/img/sdk_new_integration.png)

- Complete form fields:
    - Integration type:
        - datafold-sdk
    - Repository
    - Datasource
    - Name
- Save

![](/img/sdk_config.png)

- Select the new orchestion from the Integrations page and make note of the config id.

![](/img/sdk_config_id.png)

- Now you will see an added Datafold check on new pull requests:

![](/img/sdk_pr_example.png)

In order to complete the integration, the next step is to let Datafold know which tables to diff within your CI process.

## Bash sdk example

- Set required environment variables
    - This can be done directly or in your CI provider's variables section

```bash
export DATAFOLD_APIKEY=tnQrPAyIHquhx4x9LJdOHC28waU1P0FdCvabcabc
export DATAFOLD_HOST=https://datafold.company.io
```

```bash
datafold ci submit \
    --ci-config-id 13 \
    --pr-num 6 <<- EOF
[{
        "prod": "INTEGRATION.BEERS.BEERS",
        "pr": "INTEGRATION.BEERS_DEV.BEERS",
        "pk": ["BEER_ID"]
}]
EOF
Successfully started a diff under Run ID 401
```

:::note
The "prod", "pr", and "pk" key values will need to be variables if the goal is running dynamic tables for each PR. For example, it might make sense to create a list of ** [changed files](https://github.com/marketplace/actions/changed-files) ** in a previous step, and complete multiple diffs using a file naming convention.
:::


## Python sdk example

```python
from datafold_sdk.sdk.ci import run_diff, CiDiff
run_id = run_diff(
   host="https://datafold.company.io",
   api_key="tnQrPAyIHquhx4x9LJdOHC28waU1P0FdCvabcabc",
   ci_config_id=13,
   pr_num=6,
   diffs=[
     CiDiff(
       prod='INTEGRATION.BEERS.BEERS',
       pr='INTEGRATION.BEERS_DEV.BEERS',
       pk=["BEER_ID"]
     )
   ]
)

print(f"Successfully started a diff under Run ID {run_id}")
```

:::note
The prod, pr, and pk parameters should be set by variables if the goal is running dynamic tables for each PR. For example, it might make sense to create a list of ** [changed files](https://github.com/marketplace/actions/changed-files) ** in a previous step, and complete multiple diffs using a file naming convention.
:::

## Examples by orchestrator
```mdx-code-block
import DocCardList from '@theme/DocCardList';

<DocCardList />
```
