---
sidebar_position: 2
title: GitLab CI
---
* [Basic Config](gitlab_ci.md#basic-config)
* [Advanced Config](gitlab_ci.md#advanced-config)

## Basic Config

This job runs in two scenarios, determined in the `rules:` section:
##### Deploy Production
* **When?**
  * New commits are pushed to the default branch
* **Why?**
  * Deploy model changes into the data warehouse
##### Deploy Merge Request
* **When?**
    * A merge request is opened in Gitlab
* **Why?**
    * To understand, test, and QA changes before deploying to the data warehouse

```yml
image:
  name: ghcr.io/dbt-labs/dbt-core:1.2.2
  entrypoint: [ "" ]

run_pipeline:
  stage: deploy
  before_script:
    - pip install datafold-sdk
  script:
    - set -ex
    - dbt deps
    - dbt build --full-refresh --profiles-dir ./
  rules:
    - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
      variables:
        TYPE: "production"
        SNOWFLAKE_SCHEMA: "BEERS"
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        TYPE: "pull_request"
        SNOWFLAKE_SCHEMA: "BEERS_DEV" 
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
    * Create diffs automatically and write the details to the merge request

```yml
image:
  name: ghcr.io/dbt-labs/dbt-core:1.2.2
  entrypoint: [ "" ]

run_pipeline:
  stage: deploy
  before_script:
    - pip install datafold-sdk
  script:
    - set -ex
    - aws s3 cp s3://datafold-dbt-prod-manifest/manifest.json ./manifest.json
    - dbt deps
    - dbt build --select state:modified+ --defer --state ./ --exclude config.materialized:snapshot --profiles-dir ./
    - datafold dbt upload --ci-config-id 26 --run-type $TYPE --commit-sha $CI_COMMIT_SHA
    - dbt source freshness --profiles-dir ./
  rules:
    - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
      variables:
        TYPE: "production"
        SNOWFLAKE_SCHEMA: "BEERS"
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      variables:
        TYPE: "pull_request"
        SNOWFLAKE_SCHEMA: "BEERS_DEV" 
```
