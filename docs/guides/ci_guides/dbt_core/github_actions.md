---
sidebar_position: 2
title: GitHub Actions
---
* [Prerequisites](github_actions.md#prerequisites)
* [Basic Config](github_actions.md#basic-config)
    * [Production Job](github_actions.md#production-job)
    * [Pull Request Job](github_actions.md#pull-request-job)
* [Advanced Config](github_actions.md#advanced-config)
    * [Advanced Pull Request Job](github_actions.md#advanced-pull-request-job)

## Prerequisites
- [API Key](/integrations/orchestration/datafold-sdk#generate-a-datafold-api-key)
- [DBT Core Integration](/integrations/orchestration/dbt_core.md)
  - [CI config ID](/integrations/orchestration/dbt_core#next-steps)

## Basic Config

### Production Job

This job runs in two scenarios, defined in the `on:` section:
##### Deploy Production
* **When?**
    * New commits are pushed to the main branch
* **Why?** 
    * Deploy model changes into the data warehouse

##### Scheduled
  * **When?**
    * Every day at 2:00; for cron help - check out [crontab.guru](https://crontab.guru/)
  * **Why?**
    * Data warehouse daily refresh with new data

```yml
name: dbt prod

# Run this job on a push to the main branch or at 2AM
on:
  push:
    branches:
      - main
  schedule: # Run the pipeline at 2AM
    - cron: '0 2 * * *'

jobs:
  run:
    # This is the docker image used for the CI container
    runs-on: ubuntu-20.04

    steps:
        # Pull code from your Github repo into the container
      - name: checkout
        uses: actions/checkout@v2

        # Install Python 3.8, this is required to run dbt
      - uses: actions/setup-python@v2
        with:
          python-version: '3.8'

        # Install Python packages defined in requirements.txt (dbt and dependencies)
      - name: install requirements
        run: pip install -q -r requirements.txt

        # Install dbt packages defined in packages.yml
      - name: dbt deps
        run: dbt deps

        # Run and test dbt models
      - name: dbt build
        run: dbt build --full-refresh --profiles-dir ./
        env:
          # Secrets are pulled from the Github Actions secrets setting
          # See: https://docs.github.com/en/actions/security-guides/encrypted-secrets
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}
          SNOWFLAKE_ROLE: ${{ secrets.SNOWFLAKE_ROLE }}
          SNOWFLAKE_SCHEMA: "${{ 'ANALYTICS' }}"

        # Use the Datafold sdk to create a diff and write results to the PR
      - name: submit artifacts to datafold
        run: |
          set -ex
          datafold dbt upload --ci-config-id 999 --run-type ${DATAFOLD_RUN_TYPE} --commit-sha ${GIT_SHA}
        env:
          DATAFOLD_APIKEY: ${{ secrets.DATAFOLD_APIKEY }}
          DATAFOLD_RUN_TYPE: "${{ 'production' }}"
          GIT_SHA: "${{ github.ref == 'refs/heads/master' && github.sha }}"
```

### Pull Request Job
This job runs in one scenario, again defined in the `on:` section:
* **When?**
  * New commits are pushed a branch *other than* main
* **Why?**
  * To understand, test, and QA changes before deploying to the data warehouse

```yml
name: dbt staging

# Run this job when a commit is pushed to any branch except main
on:
  pull_request:
  push:
    branches:
      - '!main'

jobs:
  run:
    # This is the docker image used for the CI container
    runs-on: ubuntu-20.04

    steps:
        # Pull code from the Github repo into the container
      - name: checkout
        uses: actions/checkout@v2

        # Install Python 3.8, this is required to run dbt
      - uses: actions/setup-python@v2
        with:
          python-version: '3.8'

        # Install Python packages defined in requirements.txt (dbt and dependencies)
      - name: install requirements
        run: pip install -q -r requirements.txt

        # Install dbt packages defined in packages.yml
      - name: dbt deps
        run: dbt deps

        # Retrieve the PR number and set it to the variable findPR
      - name: Find Current Pull Request
        uses: jwalton/gh-find-current-pr@v1.3.0
        id: findPR

        # Run and test all dbt models
      - name: dbt build
        run: dbt build --profiles-dir ./
        env:
          # Secrets are pulled from the Github Actions secrets setting
          # See: https://docs.github.com/en/actions/security-guides/encrypted-secrets
          # The findPR variable from the previous step is used to create a schema for the current PR 
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}
          SNOWFLAKE_ROLE: ${{ secrets.SNOWFLAKE_ROLE }}
          SNOWFLAKE_SCHEMA: "${{ format('{0}_{1}', 'PR_NUM', steps.findPr.outputs.pr) }}"

      - name: submit artifacts to datafold
        run: |
          set -ex
          datafold dbt upload --ci-config-id 999 --run-type ${DATAFOLD_RUN_TYPE} --commit-sha ${GIT_SHA}
        env:
          DATAFOLD_APIKEY: ${{ secrets.DATAFOLD_APIKEY }}
          DATAFOLD_RUN_TYPE: "${{ 'pull_request' }}"
          GIT_SHA: "${{ github.event.pull_request.head.sha }}"
```

## Advanced Config

### Advanced Pull Request Job
This is similar to the [pull request job](github_actions.md#pull-request-job) above, with some added features:
* [Slim CI](https://docs.getdbt.com/docs/dbt-cloud/using-dbt-cloud/cloud-enabling-continuous-integration#configuring-a-dbt-cloud-ci-job)
    * Speeds up CI by running only your changes
    * Quick primer on [state:modified](https://docs.getdbt.com/reference/node-selection/methods#the-state-method) syntax:
        * `state:modified+` run the modified model(s) and all downstream models
        * `state:+modified` run the modified model(s) and all upstream models
        * `state:modified+n` run the modified model(s) and N downstream models
* Datafold CI Diff - Link Here? Diff Core?
    * Create diffs automatically and write the details to the PR

```yml
name: dbt staging

# Run this job when a commit is pushed to any branch except main
on:
  pull_request:
  push:
    branches:
      - '!main'

jobs:
  run:
    # This is the docker image used for the CI container
    runs-on: ubuntu-20.04

    steps:
        # Pull code from the Github repo into the container
      - name: checkout
        uses: actions/checkout@v2

        # Install Python 3.8, this is required to run dbt
      - uses: actions/setup-python@v2
        with:
          python-version: '3.8'

        # Install Python packages defined in requirements.txt (dbt and dependencies)
      - name: install requirements
        run: pip install -q -r requirements.txt

        # Install the Datafold sdk via pip
      - name: install datafold-sdk
        run: pip install -q datafold-sdk

        # Install dbt packages defined in packages.yml
      - name: dbt deps
        run: dbt deps

        # Retrieve the PR number and set it to the variable findPR
      - name: Find Current Pull Request
        uses: jwalton/gh-find-current-pr@v1.3.0
        id: findPR

        # Get the latest manifest.json from production
        # The manifest.json is used for dbt state comparison and Datafold's diff comparison
      - name: Grab production manifest from S3
        run: |
          aws s3 cp s3://datafold-dbt-prod-manifest/manifest.json ./manifest.json
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: us-west-2

        # Run and test modified dbt models and models downstream of them
        # Otherwise defer to production for unmodified models
      - name: dbt build
        run: dbt build --select state:modified+ --defer --state ./ --exclude config.materialized:snapshot --profiles-dir ./
        env:
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}
          SNOWFLAKE_ROLE: ${{ secrets.SNOWFLAKE_ROLE }}
          SNOWFLAKE_SCHEMA: "${{ format('{0}_{1}', 'PR_NUM', steps.findPr.outputs.pr) }}"

        # Use the Datafold sdk to create a diff and write results to the PR
      - name: submit artifacts to datafold
        run: |
          set -ex
          datafold dbt upload --ci-config-id 999 --run-type ${DATAFOLD_RUN_TYPE} --commit-sha ${GIT_SHA}
        env:
          DATAFOLD_APIKEY: ${{ secrets.DATAFOLD_APIKEY }}
          DATAFOLD_RUN_TYPE: "${{ 'pull_request' }}"
          GIT_SHA: "${{ github.event.pull_request.head.sha }}"

        # Run dbt's source freshness tests
      - name: dbt source freshness
        run: dbt source freshness --profiles-dir ./
        env:
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}
          SNOWFLAKE_ROLE: ${{ secrets.SNOWFLAKE_ROLE }}
          SNOWFLAKE_SCHEMA: "${{ format('{0}_{1}', 'PR_NUM', steps.findPr.outputs.pr) }}"
```

### Skip Diffing for Draft Pull Requests

In the example `dbt_staging` job provided in the [Advanced Pull Request Job](#advanced-pull-request-job) section, once a pull request is opened, _each subsequent push_ will trigger the `dbt_staging` job. If an open pull request is a work-in-progress, running staging on each push can become noisy and expensive.

By incorporating the below modifications, pull requests marked as a draft will skip the staging job. Once marked as a draft, developers can push commits until the pull request is marked ready for review, at which point the staging job will run.

```yml
name: dbt staging

on:
  pull_request:
    types:
    - opened
    - reopened
    - synchronize
    - ready_for_review
  push:
    branches:
      - '!main'

jobs:
  run:
    runs-on: ubuntu-20.04
    if: ${{ !github.event.pull_request.draft }}
```