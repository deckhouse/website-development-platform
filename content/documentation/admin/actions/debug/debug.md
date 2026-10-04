---
title: Debug
weight: 10
---

Debug runs a debugging action. It performs a specified number of wait cycles, writes arbitrary data to the log, and returns the request in the action response.

### Request example

```yaml
sleep_time: 1
sleep_count: 3
extra:
  example_key: example_value
```

### Request specification

| Name            | Required | Description                                                                                       | Default value |
| --------------- | -------- | ---------------------------------------------------------------------------------------------------- | ----------------------- |
| sleep_time      | No       | Duration of one wait cycle, in seconds                                                                | `0`                     |
| sleep_count     | No       | Number of wait cycles                                                                                 | `0`                     |
| extra           | No       | An arbitrary set of key-value pairs that is written to the log and returned in the action's response  | -                       |

With `0` values, the action finishes immediately. Negative values are not allowed: `sleep_time and sleep_count must not be negative`.

### Response

The action returns the whole request: the `sleep_time`, `sleep_count` and `extra` fields with their values.
Use them in templates of subsequent actions, for example `{{ .response.extra.example_key }}`.
