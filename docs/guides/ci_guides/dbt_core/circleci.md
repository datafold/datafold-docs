---
sidebar_position: 1
title: CircleCI
---
* [Basic Config](circleci.md#basic-config)
* [Advanced Config](circleci.md#advanced-config)


## Basic Config

This job runs in two scenarios, determined in the `if [ -z "${CIRCLE_PULL_REQUEST##*/}" ]` statement:
##### Deploy Production
* **When?**
    * New commits are pushed to the default branch
* **Why?** 
    * Deploy model changes into the data warehouse
##### Deploy Pull Request
* **When?**
    * A pull request is opened in the external repository
* **Why?** 
    * To understand, test, and QA changes before deploying to the data warehouse


```yml
version: 2.1

jobs:
  beers-ci:
    docker:
      - image: cimg/python:3.8
    steps:
      - checkout
      - run:
          name: "Install DBT"
          command: pip install -q 'dbt-core==1.2.2' 'dbt-snowflake'
      - run:
          name: "Install DBT Dependencies"
          command: dbt deps
      - run:
          # CircleCI does not support separate jobs/workflows for pull request vs. merges
          # Make sure to enable "only build pull requests" https://circleci.com/docs/oss#only-build-pull-requests
          # This will run CI only for pull requests and commits to the default branch
          # https://discuss.circleci.com/t/is-it-possible-to-run-workflows-when-github-prs-are-opened/38106
          # Add environment variables in the CircleCI interface (Ex: SNOWFLAKE_ACCOUNT, SNOWFLAKE_USER, DATAFOLD_API_KEY)
          name: "Run dbt"
          command: |
            if [ -z "${CIRCLE_PULL_REQUEST##*/}" ]
            then
              echo Setting variables for a merge to main
              export SNOWFLAKE_SCHEMA=BEERS_CIRCLE_CI
              export DATAFOLD_RUN_TYPE=production
            else
              echo Setting variables for a pull request
              export SNOWFLAKE_SCHEMA=BEERS_DEV_CIRCLE_CI
              export DATAFOLD_RUN_TYPE=pull_request
            fi
            set -ex
            dbt deps
            dbt build --full-refresh --profiles-dir ./
workflows:
  beers-ci-workflow:
    jobs:
      - beers-ci
```

## Advanced Config

### Advanced Merge Request Job
This is similar to the [pull request job](github_actions.md#pull-request-job) above, with some added features:
* [Slim CI](https://docs.getdbt.com/docs/dbt-cloud/using-dbt-cloud/cloud-enabling-continuous-integration#configuring-a-dbt-cloud-ci-job)
    * Speeds up CI by running only your changes
    * Quick primer on [state:modified](https://docs.getdbt.com/reference/node-selection/methods#the-state-method) syntax:
        * `state:modified+` run the modified model(s) and all downstream models
        * `state:+modified` run the modified model(s) and all upstream models
        * `state:modified+n` run the modified model(s) and N downstream models
* Datafold CI Diff - Link Here? Diff Core?
    * Create diffs automatically and write the details to the pull request

```yml
version: 2.1

jobs:
  beers-ci:
    docker:
      - image: cimg/python:3.8
    steps:
      - checkout
      - run:
          name: "Install DBT"
          command: pip install -q 'dbt-core==1.2.2' 'dbt-snowflake' 'datafold-sdk'
      - run:
          # CircleCI does not support separate jobs/workflows for pull request vs. merges
          # Make sure to enable "only build pull requests" https://circleci.com/docs/oss#only-build-pull-requests
          # This will run CI only for pull requests and commits to the default branch
          # https://discuss.circleci.com/t/is-it-possible-to-run-workflows-when-github-prs-are-opened/38106
          # Add environment variables in the CircleCI interface (Ex: SNOWFLAKE_ACCOUNT, SNOWFLAKE_USER, DATAFOLD_API_KEY)
          name: "Run dbt and datafold"
          command: |
            if [ -z "${CIRCLE_PULL_REQUEST##*/}" ]
            then
              echo Setting variables for a merge to main
              export SNOWFLAKE_SCHEMA=BEERS_CIRCLE_CI
              export DATAFOLD_RUN_TYPE=production
            else
              echo Setting variables for a pull request
              export SNOWFLAKE_SCHEMA=BEERS_DEV_CIRCLE_CI
              export DATAFOLD_RUN_TYPE=pull_request
            fi
            set -ex
            aws s3 cp s3://datafold-dbt-prod-manifest/manifest.json ./manifest.json
            dbt deps
            dbt build --select state:modified+ --defer --state ./ --exclude config.materialized:snapshot --profiles-dir ./
            datafold dbt upload --ci-config-id 99999 --run-type ${DATAFOLD_RUN_TYPE} --target-folder ./target/ --commit-sha ${CIRCLE_SHA1}
            dbt source freshness --profiles-dir ./
workflows:
  beers-ci-workflow:
    jobs:
      - beers-ci
```
