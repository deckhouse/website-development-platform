---
title: Wait
weight: 10
---

Wait pauses execution for a specified number of seconds and can optionally add a random duration (jitter). It is intended for use in processes as a delay element, including while waiting for the results of a previous action to be applied.

### Request example

```yaml
duration_seconds: 10
max_jitter_seconds: 0
description: "Waiting for release"
```

### Request specification

| Name                  | Required | Description                                                                        | Default value          |
| --------------------- | -------- | ------------------------------------------------------------------------------------- | ---------------------- |
| duration_seconds      | No       | Base wait duration in seconds, from `0` to `86400` (24 hours)                           | `0`                    |
| max_jitter_seconds    | No       | Maximum random addition to the wait time in seconds (0–N). 0 disables jitter            | `0`                    |
| description           | No       | Description shown in the logs and the action's response                                | -                      |

The total wait time, including jitter, does not exceed 86400 seconds.

### Response

The action returns the following fields:

| Name                     | Description                                                   |
| ------------------------ | ------------------------------------------------------------- |
| `duration_seconds`       | Base wait duration from the request, in seconds               |
| `applied_jitter_seconds` | Random addition applied to this run, in seconds               |
| `total_seconds`          | Actual wait duration, in seconds                              |
| `description`            | Description from the request                                  |

For example, `{{ .response.total_seconds }}` returns the actual wait duration.
