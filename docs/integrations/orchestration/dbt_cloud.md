---
sidebar_position: 1
title: dbt Cloud
description: Integrate Datafold with dbt Cloud jobs
---
# dbt Cloud

### Prerequisites
- To configure dbt Cloud, you must first connect a [Data Source](integrations/data_warehouses/dw_overview.md) and connect a [GitHub](/integrations/git/github.md) or [GitLab](/integrations/git/gitlab.md) account.
- To access the dbt API Datafold requires a dbt **Team** account or higher.
- You will need either a [Service Token](https://docs.getdbt.com/docs/dbt-cloud-apis/service-tokens) or a [User Token](https://docs.getdbt.com/docs/dbt-cloud-apis/user-tokens):
    - **Service Token (Recommended):** 
        - Navigate to **Account Settings -> Service Tokens -> + New Token** <br/><br/>
            ![](../../../static/img/dbt_cloud_add_service_token.png) <br/><br/>
            
            - Add a Token Name
            - Add a Permission Set <br/><br/>
                ![](../../../static/img/dbt_cloud_add_service_token_permission.png) <br/><br/>
                - Permission Set: Member
                - Project: All Projects, or check only the projects to use with Datafold
                - Save <br/><br/>
                ![](../../../static/img/dbt_cloud_service_token.png) <br/><br/>
    - **User Token:**
        - Navigate to **Your Profile -> API Access**
            - Copy
- In dbt Cloud, configure a production job and a Pull Request job, and [set up dbt Cloud CI](https://docs.getdbt.com/docs/deploy/cloud-ci-job) so that your Pull Request job runs when you open or update a Pull Request.

### Connecting dbt Cloud

* To set up dbt Cloud begin by navigating to **Admin** -> **Settings** -> **Orchestration**. Here you will click on **Add New Integration** to enter your dbt Cloud details. 

![](../../../static/img/dbt_cloud_setup.png)

### Configure dbt Cloud on Datafold
To complete the setup you'll connect your dbt Cloud account with an API key and specify settings for your dbt runs.

Complete the configuration by specifying the following fields:

| Field Name      | Description |
| ----------- | ----------- |
| Repository | Select the repository that generates the webhooks and where pull / merge requests will be raised. |
| Data Source | Select the data source where the code that is changed in the repository will run.|
| Name | An identifier used in Datafold to identify this CI configuration. |
| API Key | This is the token created [above](/docs/integrations/orchestration/dbt_cloud.md#prerequisites). |
| Account name  | This becomes selectable when a valid API key is filled in. After that, select your account to use. |
| Job that builds production tables | This becomes selectable after a valid API key is filled in. Select the job that builds production tables. |
| Skip for pull/merge requests | When selected, the Datafold CI pipeline won't be run on pull/merge requests. |
| Job that builds pull requests  | This becomes selectable after a valid API key is filled in. Select the job that builds pull requests. |
| Primary key tag | See [dbt Integration](./dbt_adv_config.md). This should be set to a value such as `primary-key` which will also be used in your dbt project configuration (yaml files and config blocks). |
| Base branch commit selection strategy | Select "Merge Base" to compare your PR to the commit in the main branch that is defined as part of the PR. Select "Latest" to compare your PR branch to the latest commit in the main branch.  |
| Sync metadata on every push to production | When selected, will sync the metadata from the dbt run with Datafold every time a push happens on the default branch.|
| Sync metadata on a schedule | Set a schedule to synchronize the dbt metadata (columns and table descriptions, tags, owners, etc), use this when you run the production job on a schedule. |
| Files to ignore | If defined, the files matching the pattern will be ignored in the PRs. The pattern uses the syntax of `.gitignore`. Excluded files can be re-included by using the negation; re-included files can be later re-excluded again to narrow down the filter. For example, to exclude everything except the `/dbt` folder, but not the dbt `.md` files, do:`*!dbt/*dbt/*.md`.|
| Mark the CI check as failed on errors | If the checkbox is disabled, the errors in the CI runs will be reported back to GitHub/GitLab as successes, to keep the check "green" and not block the PR/MR. By default (enabled), the errors are reported as failures and may prevent PR/MRs from being merged. |
| Require 'datafold' label to be present on pull request | When this is selected, the Datafold CI process will only run when the 'datafold' label has been applied. This label needs to be created manually in GitHub or GitLab, and the title or name must match 'datafold' exactly. |
| Sampling tolerance | The tolerance to apply in sampling for all data diffs. |
| Sampling confidence | The confidence to apply when sampling. |
| Sampling Threshold | Sampling will be disabled automatically if tables are smaller than specified threshold. If unspecified, default values will be used depending on the Data Source type. |

Click save. Now that you've set up the integration, Datafold will diff your impacted tables whenever you push commits to a PR. A summaryof the diff will appear in GitHub, and detailed results will appear in the Datafold app.
