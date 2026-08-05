---
title: Process store
description: Update rules, nested paths, write operations, loop context, and viewing the store during a process run.
weight: 20
---

The process store is a JSON object shared by a single process run. Tasks use it to pass data to each other: identifiers, arrays for loop iteration, intermediate results, and generated text.

Writing is done **only via rules** in the configuration of actions and "Template" elements. Reading is done via Go templates `{{ .store.<path> }}` in the request body, gateway conditions, and other fields that support templating.

## Data model

- **One store per run** — each process instance has its own store; it is preserved between steps and available when viewing the run.
- **Nested paths** — keys are specified as dot-paths without a leading dot: `notification.module_name`, `ctx.job.id`. Intermediate objects are created automatically.
- **Array indices** — a path can reference an array element: `items[0].status`, `branches[2].name`.
- **Value types** — depending on the operation, strings, numbers, objects, and JSON arrays can end up in the store. The "Write string" and "Append string" operations always work with strings; operations with the JSON suffix store the parsed structure.

{{< alert level="info" >}}
The service key `_loop` is populated by the process engine while a loop is running. Do not write to `_loop` manually using action rules — use it for reading in templates only.
{{< /alert >}}

## Store update rules

Rules are configured in the action: "Configuration" → "Update" → "Process store update". They run **after the action completes successfully**, **in list order**. Each subsequent rule sees the changes made by the previous rules of the same action.

In the process task configuration, rules are shown in read-only mode; to change them, open the action from the task's side panel.

### Rule fields

| Field | Required | Description |
| ---- | -------------- | -------- |
| Condition | No | A Go template. An empty value means the rule always runs. After rendering, the result must be `true`, `false`, `1`, or `0`; otherwise the rule is skipped |
| Operation | Yes | The way the value at the "Target" path is changed |
| Target | Yes | Dot-path in the store, without a leading dot |
| Source | Yes* | Go template for the value. Not used for the "Delete" operation |

### Operations

| Operation | Result |
| -------- | --------- |
| Write string | Replaces the value at the path with the text string from the source; JSON is not parsed |
| Write JSON | Replaces the value with the parsed JSON (object, array, number, boolean) from the source template |
| Append string | Appends text to the existing string at the path; if the key doesn't exist, a string is created. Not applicable to numbers and arrays |
| Append JSON | Adds a single JSON item to the array at the path; if the array doesn't exist, an empty array is created |
| Merge JSON (shallow) | Merges the JSON object from the source with the object at the path: top-level keys are overwritten, nested objects are replaced entirely |
| Delete | Deletes the key at the path |

For the **Write JSON**, **Append JSON**, and **Merge JSON** operations, it is convenient to use the `toJSON` function together with a direct reference to a response field, e.g. `{{ .response.items }}` — the platform will substitute the value without extra serialization.

### Rule examples

**Save an identifier from the action's response:**

| Condition | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| | Write string | `deploy.job_id` | `{{ .response.id }}` |

**Write an array for a subsequent loop:**

| Condition | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| | Write JSON | `notification.engagements` | `{{ toJSON .response.engagements }}` |

**Append a string to a log:**

| Condition | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| | Append string | `run.log` | `step {{ .store._loop.index }}: ok\n` |

**Update only part of an object:**

| Condition | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| | Merge JSON (shallow) | `meta` | `{{ toJSON (dict "status" "ready" "updated_by" .entity.slug) }}` |

**Run a rule only on the first loop iteration:**

| Condition | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| `{{ .store._loop.first }}` | Write string | `run.started_at` | `{{ now }}` |

**Delete a temporary key after use:**

| Condition | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| | Delete | `temp.payload` | |

### "Store paths" panel

The action's rule editor has a **Store paths** panel: it shows the loop context keys (`_loop.item`, `_loop.index`, etc.).

## Reading from the store

In process templates, use:

```go
{{ .store.<path> }}
```

For nested fields, the path matches the "Target" in the write rules:

```go
{{ .store.notification.module_name }}
{{ .store.items[0].id }}
```

Process launch parameters are available in a separate context:

```go
{{ .process.<parameter_identifier> }}
```

For more information on templating contexts, see [Templating](../../../user/templating/).

{{< alert level="warning" >}}
If a key is not present in the store (the action has not yet run, the rule didn't fire, or the condition was false), the template `{{ .store.<path> }}` will cause the step to fail with an error. Make sure preceding tasks have written the required data.
{{< /alert >}}

## Loop context `_loop`

While the loop body is running, the `_loop` object is available in the store:

| Key | Description |
| ---- | -------- |
| `_loop.item` | The current collection item ("By collection" mode) or the iteration number, starting from 1 ("Fixed number" mode) |
| `_loop.index` | The iteration index, starting from 0 |
| `_loop.total` | The total number of iterations |
| `_loop.first` | `true` on the first iteration |
| `_loop.last` | `true` on the last iteration |
| `_loop.parent` | The parent loop's context, for nested loops |
| `_loop.loop_element_uuid` | The UUID of the "Loop" element |

Examples in templates:

```go
{{ .store._loop.item.name }}
{{ .store._loop.item.scan_type }}
{{ .store._loop.index }}
```

In exclusive gateway conditions inside a loop:

```go
{{ eq .store._loop.item.branch .store.current_branch_tag }}
```

## Looping over a collection from the store

The "Loop" element supports two modes (the "Loop mode" field):

1. **Fixed number** — the body runs a specified number of times (1–10000). `_loop.item` holds the iteration number (1, 2, 3…).
1. **By collection** — when entering the loop, the "Collection template" Go template is evaluated; each element of the resulting **JSON array** is one iteration.

### Collection template

The recommended approach is a path to an array already written to the store by a previous task:

```go
{{ .store.notification.engagements }}
```

An alternative is to build the array in the template:

```go
{{ toJSON .store.modules }}
```

**Edge cases:**

- An empty array or a missing key — 0 iterations, execution follows the "Loop exit" connection; a warning appears in the process log.
- A value at the path that is not a JSON array — a validation or loop execution error.

### Scenario: iterating over a list from an API

1. **Task** "Get engagements" — a **Write JSON** rule, target `notification.engagements`, source `{{ toJSON .response.engagements }}`.
1. **Loop**, **By collection** mode, collection template `{{ .store.notification.engagements }}`.
1. **Task** in the loop body — use `{{ .store._loop.item.id }}`, `{{ .store._loop.item.name }}` in the request body.
1. **Exclusive gateway** in the body — a condition based on an item field, e.g. left value `{{ .store._loop.item.status }}`, right value `active`.
1. **Loop exit** connection — continuation after iterating over all elements.

## The Template element

The "Template" element evaluates a Go template as the process runs and writes a **string** to the store. JSON in the result is not parsed — the text is stored in the store as-is.

| Field | Description |
| ---- | -------- |
| Template body | A Go template; `{{ .store.* }}`, `{{ .process.* }}` are available, and, inside a loop, `{{ .store._loop.* }}` |
| Store key | The destination dot-path, e.g. `rendered_message` or `notification.summary` |
| Format hint | A hint used when viewing the run (`text`, `markdown`, `html`, `json`); does not affect the write |

Example body:

```go
## Report for {{ .store._loop.item.name }}

Status: {{ .store._loop.item.status }}
Owner: {{ .store._loop.item.owner }}
```

Subsequent tasks read the result via `{{ .store.rendered_message }}` or the corresponding path.

## Viewing the store during execution

On the process run visualization screen, open the **Store** tab:

- **Tree** — navigate nested keys; the type and value of the selected node are shown on the right.
- **JSON** — the full contents of the store in JSON format.

You can copy a value, refresh the data, and download the run's JSON (button on the run dialog panel).

The task's side panel on the run diagram shows the selected action's **Process store update rules** — useful for comparing the store's actual contents against the configuration.

## Process log and debugging

The visualization panel provides:

- an **Execution log** tab (including in a separate window or below the diagram);
- **Download log** — the run's text log;
- messages about an empty loop collection and skipped store rules.

If a template error occurs, or invalid JSON is used in a rule with a JSON operation, that rule may be skipped (with a warning in the log), while the other rules of the same action continue to run.

## Related sections

- [Overview](overview/) — diagram elements, running, and management.
- [Updating the store from actions](../../actions/overview/#process-store-update) — where the rules are enabled.
- [Templating](../../../user/templating/#process-store) — Go template syntax and contexts.
