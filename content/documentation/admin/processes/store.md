---
title: Process store
description: Update rules, nested paths, write operations, loop context, and viewing the store during a process run.
weight: 20
params:
  relatedLinks:
    - title: "Processes overview"
      url: ../overview/
    - title: "Updating the process store from actions"
      url: ../../actions/overview/#process-store-update
    - title: "Templating: process store"
      url: ../../../user/templating/#process-store
---

The process store is a JSON object shared by a single process run.
Tasks use it to pass data to each other: identifiers, arrays for loop iteration, intermediate results, and generated text.

Writing is done **only via rules** in the configuration of actions and "Template" elements.
Reading is done via Go templates `{{ .store.<PATH> }}` in the request body, gateway conditions, and other fields that support templating.

## Data model

The process store follows these rules:

- **One store per run**: Each process instance has its own store; it is preserved between steps and available when viewing the run.
- **Nested paths**: Keys are specified as dot-paths without a leading dot: `notification.module_name`, `ctx.job.id`.
  Intermediate objects are created automatically.
- **Array indices**: A path can reference an array element: `items[0].status`, `branches[2].name`.
- **Value types**: Depending on the operation, strings, numbers, objects, and JSON arrays can end up in the store.
  The "Set string" and "Append string" operations always work with strings; operations with the JSON suffix store the parsed structure.

{{< alert level="info" >}}
The service key `_loop` is populated by the process engine while a loop is running.
Do not write to `_loop` manually using action rules — use it for reading in templates only.
{{< /alert >}}

## Store update rules

Rules are configured in the action, on the "Update" tab, in the "Updating process store" section.
They run **after the action completes successfully**, **in list order**.
Each subsequent rule sees the changes made by the previous rules of the same action.

In the process task configuration, rules are shown in read-only mode; to change them, open the action from the task's side panel.

### Rule fields

Each store update rule has the following fields:

| Name  | Required | Description |
| ---- | -------------- | -------- |
| When | No | A Go template. An empty value means the rule always runs. After rendering, the result must be `true`, `false`, `1`, or `0`; otherwise the rule is skipped |
| Operation | Yes | The way the value at the "Target" path is changed |
| Target | Yes | Dot-path in the store, without a leading dot |
| Source | Yes* | Go template for the value. Not used for the "Delete" operation |

### Operations

The following operations are available:

| Operation | Result |
| -------- | --------- |
| Set string | Replaces the value at the path with the text string from the source; JSON is not parsed |
| Set JSON | Replaces the value with the parsed JSON (object, array, number, boolean) from the source template |
| Append string | Appends text to the existing string at the path; if the key doesn't exist, a string is created. Not applicable to numbers and arrays |
| Append JSON | Adds a single JSON item to the array at the path; if the array doesn't exist, an empty array is created |
| Merge JSON (shallow) | Merges the JSON object from the source with the object at the path: top-level keys are overwritten, nested objects are replaced entirely |
| Delete | Deletes the key at the path |

For the **Set JSON**, **Append JSON**, and **Merge JSON** operations, the source must render valid JSON: pass the value through `toJSON`, for example, `{{ toJSON .response.items }}`.
If the source is a single path, such as `{{ .response.items }}`, the portal substitutes the value at this path without serialization.

### Rule examples

**Save an identifier from the action's response:**

| When | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| — | Set string | `deploy.job_id` | `{{ .response.id }}` |

**Write an array for a subsequent loop:**

| When | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| — | Set JSON | `notification.engagements` | `{{ toJSON .response.engagements }}` |

**Append a string to a log:**

| When | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| — | Append string | `run.log` | `step {{ .store._loop.index }}: ok\n` |

**Update only part of an object:**

| When | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| — | Merge JSON (shallow) | `meta` | `{{ toJSON (dict "status" "ready" "updated_by" .entity.slug) }}` |

**Run a rule only on the first loop iteration:**

| When | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| `{{ .store._loop.first }}` | Set string | `run.started_at` | `{{ now }}` |

**Delete a temporary key after use:**

| When | Operation | Target | Source |
| ------- | -------- | ---- | -------- |
| — | Delete | `temp.payload` | — |

### "Loop context" panel

The action's rule editor has a **Loop context** panel: it shows the loop context keys, such as `_loop.item` and `_loop.index`.
Click a key to insert it into a rule.

## Reading from the store

In process templates, use:

```go-text-template
{{ .store.<PATH> }}
```

For nested fields, the path matches the "Target" in the write rules:

```go-text-template
{{ .store.notification.module_name }}
```

Read an array element with the `index` function:

```go-text-template
{{ (index .store.items 0).id }}
```

An index in square brackets, such as `items[0].id`, works only in write paths, for example in the "Target" field of write rules.
In a Go template it fails the step with an error.

Process launch parameters are available in a separate context:

```go-text-template
{{ .process.<parameter_identifier> }}
```

For more information on templating contexts, see [Templating](../../../user/templating/).

{{< alert level="warning" >}}
If a key is not present in the store (the action has not yet run, the rule didn't fire, or the condition was false), the template `{{ .store.<PATH> }}` will cause the step to fail with an error.
Make sure preceding tasks have written the required data.
{{< /alert >}}

## Loop context _loop

While the loop body is running, the `_loop` object is available in the store:

| Key | Description |
| ---- | -------- |
| `_loop.item` | The current collection item ("Foreach collection" mode) or the iteration number, starting from 1 ("Fixed count" mode) |
| `_loop.index` | The iteration index, starting from 0 |
| `_loop.total` | The total number of iterations |
| `_loop.first` | `true` on the first iteration |
| `_loop.last` | `true` on the last iteration |
| `_loop.parent` | The parent loop's context, for nested loops |
| `_loop.loop_element_uuid` | The UUID of the "Loop" element |

Examples in templates:

```go-text-template
{{ .store._loop.item.name }}
{{ .store._loop.item.scan_type }}
{{ .store._loop.index }}
```

In exclusive gateway conditions inside a loop:

```go-text-template
{{ eq .store._loop.item.branch .store.current_branch_tag }}
```

## Looping over a collection from the store

The "Loop" element supports two modes (the "Loop mode" field):

1. **Fixed count**: The body runs a specified number of times (1–10000).
   `_loop.item` holds the iteration number (1, 2, 3…).
1. **Foreach collection**: When entering the loop, the "Collection template" Go template is evaluated; each element of the resulting **JSON array** is one iteration.
   The array can contain up to 10,000 items.

### Collection template

The recommended approach is a path to an array already written to the store by a previous task:

```go-text-template
{{ .store.notification.engagements }}
```

An alternative is to build the array in the template:

```go-text-template
{{ toJSON .store.modules }}
```

**Edge cases:**

- An empty array or a missing key: 0 iterations, execution follows the "Exit" connection, and a warning appears in the process log.
- A string containing a JSON array, for example, the result of a "Set string" rule with the `{{ toJSON .response.items }}` source: The string is parsed as an array.
- A value at the path that is not a JSON array: A validation or loop execution error.

### Scenario: iterating over a list from an API

To iterate over a list returned by an API, configure the process as follows:

1. **Task** "Get engagements" — a **Set JSON** rule, target `notification.engagements`, source `{{ toJSON .response.engagements }}`.
1. **Loop**, **Foreach collection** mode, collection template `{{ .store.notification.engagements }}`.
1. **Task** in the loop body — use `{{ .store._loop.item.id }}`, `{{ .store._loop.item.name }}` in the request body.
1. **Exclusive gateway** in the body — a condition based on an item field, for example, left value `{{ .store._loop.item.status }}`, right value `active`.
1. **Exit** connection — continuation after iterating over all elements.

## The Template element

The "Template" element evaluates a Go template as the process runs and writes a **string** to the store.
JSON in the result is not parsed — the text is stored in the store as-is.

| Name  | Description |
| ---- | -------- |
| Template body | A Go template; `{{ .store.* }}`, `{{ .process.* }}` are available, and, inside a loop, `{{ .store._loop.* }}` |
| Output store key | The destination dot-path, for example, `rendered_message` or `notification.summary` |
| Format hint | A hint used when viewing the run (`text`, `markdown`, `html`, `json`); does not affect the write |

Example body:

```go-text-template
## Report for {{ .store._loop.item.name }}

Status: {{ .store._loop.item.status }}
Owner: {{ .store._loop.item.owner }}
```

Subsequent tasks read the result via `{{ .store.rendered_message }}` or the corresponding path.

## Viewing the store during execution

On the process run visualization screen, open the **Store** tab.
The **Tree** view lets you navigate nested keys and shows the type and value of the selected node on the right.
The **JSON** view shows the full contents of the store in JSON format.

You can copy a value, refresh the data, and download the run's JSON (button on the run dialog panel).

The task's side panel on the run diagram shows the selected action's **Process store update rules**.
Compare them with the actual contents of the store to find a rule that did not run.

## Process log and debugging

The visualization panel provides:

- An **Execution log** tab (including in a separate window or below the diagram).
- **Download log**: The run's text log.
- Messages about an empty loop collection and skipped store rules.

If a template error occurs, or invalid JSON is used in a rule with a JSON operation, that rule may be skipped (with a warning in the log), while the other rules of the same action continue to run.
