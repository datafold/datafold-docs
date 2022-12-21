---
sidebar_position: 4
title: Slim Diff
---

By default, Datafold diffs all modified models and downstream models. However, it won't make sense for all organizations to diff every downstream table every time you make a code update. Tradeoffs of time, cost, and risk must be considered.

That's why we created Slim Diff. With Slim Diff enabled, Datafold will only diff models that have changed code in your Pull Request (PR).

[IMAGE OF CONFIGURATION OPTION]
[IMAGE OF DOWNSTREAM MODELS THAT ARE NOT DIFFED]

Once Datafold has diffed only the modified models, you have the option of diffing individual downstream models right within your PR.

[IMAGE OF DOWNSTREAM MODELS WITH DIFF BUTTON]

Also within your PR, you can add the `datafold:diff-all-downstream` label that will automatically diff _all_ downstream models.

[IMAGE OF THE LABEL APPLIED]

Finally, when you have Slim Diff turned on, you might have certain models or subdirectories that you want to _always_ diff when downstream. You can think of this as an exclusion to the Slim Diff behavior, since Datafold always diffs downstream models when Slim Diff is off.

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