---
title: Store for actions in BPMN — usage guide
menuTitle: Store (BPMN)
d8Edition: ee
moduleStatus: experimental
---

This document describes how the **store** works in BPMN processes: how data is written to the store and how later actions read it.

## 1. What is the store?

The store is a **flat** key–value map **within a single process instance**. It holds only the data that is explicitly written according to action rules. Another instance of the same process has its own store.

- **One store per process instance** — each process run has its own store.
- **Flat keys** — the store has no "task slug" level; only logical keys (e.g. `projectId`, `deployJobId`).
- **Write only by rules** — data is written to the store only when configured in the action settings (Response to store rules). If there are no rules, nothing is written to the store.
- **Read via placeholders** — in action configuration (URL, headers, body, etc.) you can use Go-style templates such as `{{ .store.<key> }}`.

## 2. How data gets into the store

### Response to store rules (on the action)

In the **action** configuration, in the "Update" section, there is a block **"Write to process store"**. In it you define a list of rules:

| Field | Value | Description |
|-------|-------|-------------|
| **Source** | Go template string | Expression with context = **response** of this action. Examples: `{{ .id }}`, `{{ .name }}`, `{{ .result.projectId }}`. The template is executed once after the action completes successfully; the result (string) is written to the store. |
| **Target** | Store key | Name of the key in the process store (e.g. `projectId`, `deployJobId`). The value from Source is saved under this key. |

- If the action has **no rules** (empty list) — nothing is written to the store when the task completes successfully.
- If there are rules — after the task completes successfully, for each rule the Source template is executed with the response as context, and the result is written to the store under the Target key.

### When writing happens

1. The process task with this action completed **successfully** (action record status = Success).
2. The action has a **non-empty** list of Response to store rules.
3. For each rule: the Go template (Source) is executed with the response data; the resulting string is written to the store under the Target key.

If the same Target key is written by several tasks (or by the same action on re-run), **the value is overwritten**. When overwriting an existing key, a warning (Warn) is logged: store key already exists, data will be overwritten.

## 3. How to read from the store in other actions

In any **later** action of the same process instance, in fields that support placeholders (URL, headers, body, etc.), you can use:

- **Syntax:** `{{ .store.<key> }}`

`<key>` is the Target from the Response to store rules (e.g. `projectId`, `deployJobId`). The store holds only these flat keys.

### Examples

- URL with project ID from a previous step:

```yaml
url: "https://api.example.com/projects/{{ .store.projectId }}/deploy"
```

- Several keys from the store in one action:

```yaml
headers:
  X-Project-Id: "{{ .store.projectId }}"
  X-Job-Id: "{{ .store.deployJobId }}"
body: |
  {"ref": "{{ .store.orderRef }}"}
```

An action that **reads** from the store does not need to know which action or task wrote the value — it only needs the **key name** (Target) that is declared in the process (in the settings of actions that write to the store).

## 4. Behavior when data is missing

- If a **key** is not in the store (the task has not run yet, had no write rules, or the write did not happen), the placeholder `{{ .store.<key> }}` is replaced with an **empty string**.
- The action **does not fail** — the request is sent with an empty value in that place.

So you can safely reference keys that are not yet in the store.

## 5. Summary

| Question | Answer |
|----------|--------|
| **Where is the store** | In the process instance data. One store per instance. |
| **What is in the store** | Only key–value pairs written by Response to store rules (flat set of keys). |
| **How to write** | In the **action** settings, add "Response to store" rules: Source — Go template over response (e.g. `{{ .id }}`), Target — store key (e.g. `projectId`). After the task completes successfully, values are written to the store according to these rules. |
| **How to read** | In another action's config use `{{ .store.<key> }}`, where `<key>` is one of the Targets from the rules. |
| **No rules on the action** | Nothing is written to the store when this action runs. |
| **Same key written multiple times** | The last written value remains; a warning about overwrite is logged. |
| **Key not in store** | Placeholder yields empty string; action does not fail. |

## 6. Example scenario

1. Action "Create project" is configured with Response to store rules:
   - Source: `{{ .id }}`, Target: `projectId`
   - Source: `{{ .name }}`, Target: `projectName`
2. A task with this action completed successfully; response was `{ "id": "123", "name": "My Project" }`.
3. The instance store now has: `projectId` = "123", `projectName` = "My Project".
4. The next task in the process uses an action with URL:  
   `https://api.example.com/projects/{{ .store.projectId }}/deploy`  
   At execution time `projectId` = "123" is substituted.

## 7. Checking the store with the Debug action

To verify that data is written to and read from the store, you can use the built-in **Debug** action.

1. **Add a task with the Debug action** to the process (or create a process with a single such task).
2. **Debug request body** defines the data that will appear in the response. Debug puts the same fields in the response as in the request: `sleep_time`, `sleep_count`, `extra`. In `extra` you can put arbitrary keys — they are available in templates as `{{ .extra.<key> }}`.

   Example body for Debug (fields from `extra` and top level go into the response and are available in templates as `{{ .extra.projectId }}`, `{{ .sleep_time }}`, etc.):

```yaml
sleep_time: 1
sleep_count: 1
extra:
  example_key: example_value
  projectId: "test-123"
  message: "hello from debug"
```

   If the body does not contain `extra.projectId` or `extra.message`, those keys will not appear in the store (the template will yield an empty value; empty values are not written to the store).

1. **Configure "Write to process store"** for this action:
   - Rule 1: Source — `{{ .extra.projectId }}`, Target — `projectId`
   - Rule 2: Source — `{{ .extra.message }}`, Target — `debugMessage`

1. After the task completes successfully, the store will have keys `projectId` and `debugMessage`. In the **next** task of the process, in the action config use, for example:
   - URL or body: `{{ .store.projectId }}`, `{{ .store.debugMessage }}`

This way you can confirm that data from Debug is written to the store via the rules and is available to later actions.
