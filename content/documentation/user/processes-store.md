---
title: Store for actions in BPMN — usage guide
menuTitle: Store (BPMN)
d8Edition: ee
moduleStatus: experimental
---

This document describes how to use the **store** in BPMN processes: how task responses are saved under a slug and how later tasks can reference them in action configuration via placeholders.

## 1. What is the store?

The store is a key–value map **per process instance**. It holds the **response payloads** of completed tasks, each under a key equal to that task element's **slug**. Only the **action response** (the object returned by the action) is written to the store; the request body is not.

- **One store per process instance:** each run of a process has its own store. Another instance of the same process has a different store.
- **Key = task slug:** when a task finishes successfully, its action response is stored under the slug of that task element.
- **Read via placeholders:** in action configuration (URL, headers, body, etc.) you use Go-style template placeholders to read from the store.

## 2. How it works

### 2.1. Task element slug

Each task element in the process has a **slug** (e.g. `create-project`, `deploy-step`). The slug must be unique within the process and is used as the store key for that task's response.

### 2.2. Writing to the store

When a task completes successfully, the engine saves that action's **response** into the instance store under the task's slug. No separate "store key" field is used; the slug is the key. If a **temporary response** is used, that response is not written to the store.

### 2.3. Reading from the store

When an action is prepared (e.g. before it runs), the current store is passed as process context. Any string field in the action configuration that supports placeholders can use store placeholders. Missing slugs or fields render as **empty string** and do **not** fail the action.

## 3. How to use

### 3.1. Set the slug on task elements

In the process editor, for each task element that should expose its result to later steps:

- Set the **Slug** field (e.g. `create-project`, `fetch-order`, `deploy-step`).
- Slug rules: required for tasks that need to be referenced from the store; typically 3–64 characters; use a format like `lowercase-with-hyphens`.

Slugs must be unique among task elements in the same process.

### 3.2. Reference the store in action configuration

In any **later** action (in the same process instance), use Go template placeholders:

- **Syntax:** `{{ .store.<slug>.<field> }}`
- **Nested fields:** `{{ .store.<slug>.<field>.<nested> }}`

`<slug>` is the task element slug; `<field>` (and optional `<nested>`) are keys from that task's action response.

#### Examples

- Use the ID from a "create project" task in a later request:

```yaml
url: "https://api.example.com/projects/{{ .store.create-project.id }}/deploy"
```

- Use multiple store entries in one action:

```yaml
headers:
  X-Project-Id: "{{ .store.create-project.id }}"
  X-Job-Id: "{{ .store.deploy-step.jobId }}"
body: |
  {"ref": "{{ .store.previous-step.ref }}"}
```

- Nested field (if the response is an object with nested data):

```yaml
projectId: "{{ .store.create-project.result.projectId }}"
```

### 3.3. Behavior when data is missing

- If a **slug** is not in the store (e.g. the task has not run yet or failed), `{{ .store.<slug>.<field> }}` is replaced with an **empty string**.
- If a **field** is missing in the response under that slug, the placeholder also renders as **empty string**.
- The action is **not** failed because of missing store data; only the placeholder output is empty.

You can safely reference store keys that may not exist; the request will still be sent, with empty values where the store had no data.
