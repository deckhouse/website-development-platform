---
title: Fail
weight: 20
---

Fail simulates an action execution error. It is intended for use as a debugging step in processes.

## Request example

Example of the action request body:

```yaml
fail: true
```

## Request specification

The action request body supports the following parameters:

| Name       | Required | Description                                                                        | Default value |
| ---------- | -------- | -------------------------------------------------------------------------------------- | ----------------------- |
| `fail`     | No       | If `true`, the action completes with an error; if `false`, the action completes successfully | `true`                  |
