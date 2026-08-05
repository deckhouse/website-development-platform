---
title: Wait
weight: 10
---

Wait waits for a specified number of seconds; a random addition to the duration (jitter) can optionally be applied. Intended for use in processes as a delay element, including for waiting until the results of a previous action have been applied.

### Request example

```yaml
duration_seconds: 10
max_jitter_seconds: 0
description: "Waiting for release"
```

### Request specification

| Name                  | Required | Description                                                                        | Default value          |
| --------------------- | -------- | ------------------------------------------------------------------------------------- | ---------------------- |
| duration_seconds      | Yes      | Base wait duration in seconds (0–86400, i.e. up to 24 hours)                            | -                      |
| max_jitter_seconds    | No       | Maximum random addition to the wait time in seconds (0–N). 0 disables jitter            | `0`                    |
| description           | No       | Description shown in the logs and the action's response                                | -                      |
