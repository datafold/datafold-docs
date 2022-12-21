---
sidebar_position: 4
title: Slim Diff
---

By default, Datafold diffs all modified models and downstream models. However, it won't make sense for all organizations to diff every downstream table every time you make a code update. Tradeoffs of time, cost, and risk must be considered.

That's why we created Slim Diff.

With Slim Diff enabled, Datafold will only diff models with code changes in your Pull Request (PR).

### Setting up Slim Diff

You can turn on Slim Diff in the Datafold app by navigating to Admin > Settings > Intregrations > Orchestration > [Select your Orchestration] > check the Slim Diff box. 

<br>
<img width="537" alt="Screen Shot 2022-12-20 at 9 50 03 PM" src="https://user-images.githubusercontent.com/1799931/208831392-5ff34df9-38d2-4f5f-b083-fe1410ba766a.png">
<br>

### Diffing only modified models

With this setting turned on, only the modified models will be diffed by default.

<br>
<img width="943" alt="Screen Shot 2022-12-20 at 9 58 55 PM" src="https://user-images.githubusercontent.com/1799931/208832523-c3552417-8975-4460-91ed-fd7b0df7d7b7.png">
<br>

### Diff individual downstream models

Once Datafold has diffed only the modified models, you still have the option of diffing individual downstream models right within your PR.

<br>
<img width="867" alt="Screen Shot 2022-12-20 at 10 00 17 PM" src="https://user-images.githubusercontent.com/1799931/208832659-e3cdb9d9-c468-459f-85ff-990b2a68b57c.png">
<br>

### Diff all downstream models 

You can also add the `datafold:diff-all-downstream` label within your PR, which will automatically diff _all_ downstream models.

<br>
<img width="942" alt="Screen Shot 2022-12-20 at 10 03 23 PM" src="https://user-images.githubusercontent.com/1799931/208833093-f853bde7-d12a-4b9f-b5ac-a4d8d9666076.png">
<br>

### Exclude models and directories from Slim Diff

Finally, with Slim Diff turned on, there might be certain models or subdirectories that you want to _always_ diff when downstream. You can think of this as an exclusion to the Slim Diff behavior.

Apply the `slim_diff: diff_when_downstream` meta tag to an entire folder:

```yaml
models:
  <project_name>:
    <directory_name>:      
      +materialized: view
      +meta:
        datafold:
          datadiff:
            slim_diff: diff_when_downstream
```

Apply the `slim_diff: diff_when_downstream` meta tag to an individual model:

```yaml
models:
  <project_name>:
    <directory_name>:        
      +materialized: view
      <model_name>:
        +meta:
          datafold:
            datadiff:
              slim_diff: diff_when_downstream
```

With this configuration in place, Slim Diff will prevent downstream models from being run _unless_ they have been designated as exceptions with the `slim_diff: diff_when_downstream` dbt meta tag.

<br>
<img width="962" alt="Screen Shot 2022-12-20 at 10 09 50 PM" src="https://user-images.githubusercontent.com/1799931/208833985-031a04fe-864a-4487-8a64-ec80e4c232e1.png">
<br>

As usual, once the PR has been opened, you'll still have the option of diffing individual downstream models that weren't diffed, or diffing all downstream models using the `datafold:diff-all-downstream` label.
