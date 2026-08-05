---
title: Fail
weight: 20
---

Fail emulates an action execution error. Intended for use in processes as a debugging element.

### Request example

```yaml
fail: true
```

### Request specification

| Name       | Required | Description                                                                        | Default value |
| ---------- | -------- | -------------------------------------------------------------------------------------- | ----------------------- |
| fail       | No       | If `true`, the action completes with an error; if `false`, the action completes successfully | `true`                  |
