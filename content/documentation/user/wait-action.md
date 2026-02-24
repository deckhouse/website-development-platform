---
title: Wait action
menuTitle: Wait action
d8Edition: ee
moduleStatus: stable
---

**Wait** is a built-in platform action that pauses execution for a specified number of seconds. It is used in processes and workflows when you need a pause between steps (for example, to allow time for deployment or an external service), to simulate a delay in tests, or to spread load using jitter. Unlike the Debug action, Wait is intended only for waiting and has no extra parameters.

## 1. Parameters (action body)

Parameters are set in the action **Body** field in YAML format.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `duration_seconds` | number | yes | — | Duration of the wait in seconds. From 0 to 86400 (24 hours). |
| `jitter_seconds` | number | no | 0 | Random addition: a random value from 0 to `jitter_seconds` is added to `duration_seconds`. 0 means no jitter. Cannot be negative. |
| `description` | string | no | — | Label for logs (e.g. "Wait after deploy"). Shown in logs at start and in the `description` field of a successful response. |

### Limits

- `duration_seconds`: from 0 to **86400** (24 hours). If the limit is exceeded, execution fails with an error.
- Total duration (including jitter) is also limited to 86400 seconds.

### Example body (YAML)

```yaml
duration_seconds: 10
jitter_seconds: 0
description: "Wait after deploy"
```

Minimal example (required field only):

```yaml
duration_seconds: 5
```

## 2. Overriding duration at run time

When starting the action (from an entity, from a process or workflow), you can pass the duration in **Properties**. If set, it overrides `duration_seconds` from the action body.

Two keys are supported (one is enough):

- **`duration_seconds`** — same name as in the body;
- **`wait`** — convenient when the action has a property with slug `wait` (number of seconds).

The value must be a non-negative integer (or a string with a number). If the value is negative or not numeric, the value from the body is used.

Example: body has `duration_seconds: 60`, and Properties at run time has `wait: 30` → wait will be 30 seconds.

## 3. Cancellation

The wait is split into 1-second intervals. After each interval, cancellation is checked (e.g. process stop or action run cancellation). Cancellation takes effect within about 1 second. On cancel, the action completes with an error (e.g. `context canceled`).

## 4. Response on success

The action response field contains JSON, for example:

```json
{
  "duration_seconds": 10
}
```

If `description` was set in the body, it is also included in the response:

```json
{
  "duration_seconds": 10,
  "description": "Wait after deploy"
}
```

`duration_seconds` in the response is the actual wait time in seconds (including jitter and the maximum limit).

## 5. Where to use

- **Process (BPMN):** add a task step with a Wait action to insert a pause between steps (e.g. between deploy and health check).
- **Workflow:** add a Wait action in the action chain to delay between steps.
- **Run from entity:** rarely needed; Wait is usually used inside processes and workflows.

## 6. Difference from Debug

| Aspect | Wait | Debug |
|--------|------|-------|
| Purpose | Waiting only | Debugging, tests, logs, sleep |
| Parameters | duration, jitter, description | sleep_time, sleep_count, extra |
| Override at run time | duration_seconds / wait | No |
| Time limit | Yes (24 h) | No |
| Cancellation check | Every second | After each sleep |

For pauses in processes and workflows, use **Wait**.
