---
title: Fail
weight: 20
---

Fail simulates an action execution error. It is intended for use as a debugging step in processes.

### Request example

```yaml
fail: true
```

### Request specification

| Name       | Required | Description                                                                        | Default value |
| ---------- | -------- | -------------------------------------------------------------------------------------- | ----------------------- |
| fail       | No       | If `true`, the action completes with an error; if `false`, the action completes successfully | `true`                  |
