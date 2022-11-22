---
sidebar_position: 4
title: Slim CI
description: Running only the diffs you need
---

By default, Datafold will run Data Diffs on any models that are edited in the Pull Request as well as any downstream models.

In some cases, this leads to too much diffing, which can be overwhelming and slow. That's why we've created the Slim CI configuration, which only diffs modified models. You still have the option of diffing individual downstream models, or automatically diffing all downstreams.

[ LOOM VIDEO ] 

### How it works

To enable Slim CI by default on all future commits that are pushed to GitHub or GitLab, select the Slim CI checkbox in your orchestration settings and .

[ SCREENSHOT ]

When you push a change, you'll see that Data Diffs have been run for only the modified models.

[ SCREENSHOT ]

You have the option within the PR to click and run any downstream Data Diffs.

[ SCREENSHOT ]

You can also apply the `datafold:full` label to the PR, which will automatically run all downstream Data Diffs.

[ SCREENSHOT ] 

