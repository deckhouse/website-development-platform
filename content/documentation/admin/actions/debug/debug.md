---
title: Debug
weight: 10
---

Debug runs a debug action. It lets you perform a specified number of wait cycles, as well as write arbitrary data to the log and return it in the action's response.

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
| sleep_time      | No       | Duration of one wait cycle, in seconds                                                                | `1`                     |
| sleep_count     | No       | Number of wait cycles                                                                                 | `1`                     |
| extra           | No       | An arbitrary set of key-value pairs that is written to the log and returned in the action's response  | -                       |
