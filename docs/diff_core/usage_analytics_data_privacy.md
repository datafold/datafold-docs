---
sidebar_position: 8
title: Usage Analytics & Data Privacy
---

# Usage Analytics & Data Privacy

data-diff collects anonymous usage data to help our team improve the tool and to apply development efforts to where our users need them most. 

If you do not wish to participate, the tracking can be easily disabled with one of the following methods:

* In the CLI, use the `--no-tracking` flag.

* In the config file, set `no_tracking = true` (for example, under `[run.default]`)

* If you're using the Python API:

```python
import data_diff
data_diff.disable_tracking()    # Call this first, before making any API calls

# Connect and diff your tables without any tracking
```

We capture two events, one when the data-diff run starts and one when it is finished. No user data or potentially sensitive information is or ever will be collected. The captured data is limited to:

- Operating System and Python version

- Types of databases used (postgresql, mysql, etc.)

- Sizes of tables diffed, run time, and diff row count (numbers only)

- Error message, if any, truncated to the first 20 characters.

- A persistent UUID to indentify the session, stored in `~/.datadiff.toml`