---
title: Overview
weight: 10
description: Creating and running processes — diagram elements, parameters, store, and execution management.
---

Processes are an automation mechanism for complex business scenarios that lets you build visual execution diagrams for actions, with support for conditional logic, parallel execution, and error handling.

## Key concepts

### Process elements

A process consists of various types of elements:

* "Start" — the process entry point.
* "Task" — runs a specific action.
* "Exclusive gateway" — conditional branching.
* "Parallel gateway" — merges several process branches into one.
* "Loop" — repeats part of the process a fixed number of times, or iterates over a JSON array from the store.
* "Template" — evaluates a Go template and writes the result to the process store.
* "Timer" — pauses until a specified point in time.
* "Note" — a text block.
* "Error" — immediately stops the process with the `Failed` status.
* "End" — completes the process.

## Creating a process

### Basic information

To create a process, go to "Self-service" → "Processes" and click "Create".

Fill in the following fields:

* "Name" — the process name.
* "Description" — a detailed description of the process's purpose.
* "Resource" — one or more resources for which the process can be run.
* "Owner" — the user responsible for the process.
* "Owner team" — the team responsible for the process.
* "Tags" — tags used to categorize the process.
* "Icon" — the icon displayed in the interface.

### Process configuration

The process is configured in the visual editor, on the "Configuration" tab.

{{< alert level="info" >}}
To configure a process, you must first fill in the basic fields and create it. After that, the "Configuration" and "Parameters" tabs become available.
{{< /alert >}}

#### Adding elements

To add elements to the diagram:

1. Add an element by selecting its type in the editor panel.
1. Configure the element's parameters in the side panel (action, conditions, etc.).
1. Connect the elements to each other.

#### Element types

##### Task

Each task runs a specific action previously created in "Self-service" → "Actions".

##### Exclusive gateway

An exclusive gateway lets you configure process branching based on conditions. By default, the status of the previous task is checked, and depending on it, execution follows the "success" or "failure" branch.

As conditions for an exclusive gateway, you can set either a check of the previous task's status, or a comparison of values from templates:

- To check the status of the previous task, use the construct `{{ .prev_task.status }}`.
- To check a value from the store, use `{{ .store.<path> }}`, including nested keys: `{{ .store.notification.status }}`.
- Inside a loop — fields of the current item: `{{ .store._loop.item.scan_type }}`, and the flags `{{ .store._loop.first }}` and `{{ .store._loop.last }}`.

You can check multiple conditions and combine their results using the `AND` or `OR` operator.

If the condition check passes, the process follows the "True" branch (green); if not, it follows the "False" branch (red). The "True" and "False" ports can be placed on any side of the gateway; a connection can be moved to a different port.

##### Parallel gateway

Used to merge several branches into one, based on specified conditions.

In the parallel gateway's configuration, you can set the following waiting parameters:

- Wait for all incoming elements to complete before continuing execution.
- Wait for at least one incoming element to complete before continuing execution.

You can also configure what exactly counts as "completion":

- Only successful completion of the tasks feeding into the gateway.
- Any final status of the incoming tasks (`Failed`, `Skipped`, etc.).

A parallel gateway can also be used to split branches, if connected after "Exclusive gateway" or "Loop" elements, or as a helper element to improve the readability of the process diagram.

##### Loop

The "Loop" element repeats a branch of the process either a fixed number of times, or once for each element of a JSON array from the store. Once all iterations are complete, the element connected to the loop's exit is activated.

###### Loop mode

Two modes are available:

- "Fixed number" — the "Number of iterations" field, from 1 to 10000. On each iteration, `{{ .store._loop.item }}` holds the iteration number (1, 2, 3…).
- "By collection" — the "Collection template" field: a Go template that, when entering the loop, must produce a JSON array. Typically, this is a path to an array written by a previous task: `{{ .store.notification.engagements }}`. Each array element is one iteration; the current element is available as `{{ .store._loop.item }}`.

If the array is empty or the key is missing, the loop body is not executed, and the process immediately follows the "Loop exit" connection.

For more information, see [Process store](store/#looping-over-a-collection-from-the-store).

###### Outgoing connections

Exactly two connections to different elements must originate from the "Loop" element:

1. "Loop body" — the branch built from this port is executed on each iteration.
1. "Loop exit" — execution continues along this branch after the last iteration.

The "Loop body" and "Loop exit" ports can be placed on any side of the element; a connection can be moved to a different port.

###### Usage

Typical scenarios:

- repeatedly checking the status of an external system with a fixed number of attempts;
- iterating over a list of objects from an API response: a task writes the array to the store, and a loop in "By collection" mode processes each element;
- nested loops — the parent context is available via `{{ .store._loop.parent }}`.

##### Template

The "Template" element evaluates a Go template without calling an additional action, and writes the result to the store as a string.

Configure:

- "Template body" — text with placeholders `{{ .store.* }}`, `{{ .process.* }}`, and, inside a loop, `{{ .store._loop.* }}`;
- "Store key" — the destination dot-path, e.g. `rendered_message`;
- "Format hint" — how to display the value when viewing the run (`text`, `markdown`, `html`, `json`); does not affect the store's content.

For examples and limitations, see [Process store](store/#the-template-element).

##### Timer

The "Timer" element pauses process execution until a specified point in time, after which the next element is activated.

While the process is only waiting for the timer to fire (no other active tasks), the run is placed in the "Wait" status. The run visualization panel shows a banner with the estimated resumption time.

###### Outgoing connections

Exactly one connection to the next element must originate from the "Timer" element. Connection is only possible through the left (input) and right (output) ports.

If the timer is passed through again within the same run (for example, via a loop), the trigger time is recalculated.

###### Schedule modes

In the timer's configuration, select a "Schedule":

- "Delay after entering the element" — a fixed pause from the moment the process reaches the timer. Set the "Delay (seconds)" from 1 to 1,209,600 (14 days).
- "Custom schedule" — triggers at a specified calendar time in the selected time zone (IANA, e.g. `Europe/Moscow`).

For "Custom schedule" mode, specify a "Pattern":

- "Specific day of the week" — "Day of the week", "Time of day" (hour and minute), "Time zone".
- "Specific day of each month" — "Day of month" (1–31), "Time of day", "Time zone". If the month does not have that day, the last day of the month is used.
- "Every N days" — "Every N days" (1–365), "Time of day", "Time zone". The first trigger is no earlier than N calendar days from the day the process reached the timer; upon re-entering the element, the calculation is redone.

"Time of day" is set in the selected time zone (the "Time zone" field), if it differs from the browser's time zone.

###### Usage

Typical scenarios: a pause before re-checking the status of an external system, a delayed start of the next step, waiting for a maintenance window, or a regular calendar slot.

##### Note

The "Note" element is intended for text annotations on the process diagram: it is not executed at runtime and is not connected to other elements.

In the note's configuration, you can set:

- "Text" — content with Markdown support.
- "Background color" — the fill color.
- "Text color" — the text color.

A note is always positioned behind other elements and can be used to visually highlight parts of the process. It is also the only element whose size can be changed.

##### Error

The "Error" element forcibly transitions a running process to the `Failed` status: when this element is reached, execution of all process actions is interrupted.

No connections to subsequent elements should originate from this element.

### Process parameters

The "Parameters" tab is used to configure process parameters that can be used in all actions within the process.

The configuration and use of process parameters are described in [Templating](../../user/templating/#process-parameters).

### Process store

The process store is a JSON object used to pass data between steps of a single run: identifiers, arrays, intermediate results, and text produced by the "Template" element.

For a full description of write rules, operations, loop context, and example scenarios, see [Process store](store/).

In brief:

- **One store per run** — each process instance has its own store; it can be viewed on the "Store" tab when visualizing a run.
- **Nested paths** — keys are specified with dots: `notification.module_name`, `ctx.job.id`; array indices are supported: `items[0].status`.
- **Rule-based writes** — data is written to the store after an action completes successfully (see [Process store update](../../actions/overview/#process-store-update)), or when the "Template" element runs. Available operations include writing strings and JSON, appending, merging, and deleting, plus a condition for each rule.
- **Reading** — Go templates `{{ .store.<path> }}` in action configuration and gateway conditions (see [Process store](../../user/templating/#process-store)).

If a key is not present in the store, the template `{{ .store.<path> }}` will cause the step to fail with an error.

## Running a process

### Manual run

To run a process manually:

1. Go to the entity for which the process needs to be run.
1. In the entity's menu, select "Run process".
1. Choose the desired process from the list.
1. Fill in the process parameters.
1. Click "Run".

### Launch parameters

The following are available when launching a process:

* "Common process parameters" — parameters defined in the process configuration.
* "Action parameters" — parameters for each action in the process.
* "Environment variables" — additional variables for execution.

## Execution management

### Process statuses

A process can be in one of the following statuses:

* "Created" — the process has been created but not run.
* "Running" — the process is currently running.
* "Paused" — process execution has been paused.
* "Completed" — the process completed successfully.
* "Failed" — the process finished with an error.
* "Cancelled" — process execution was cancelled.

### Management operations

The following operations are available for active processes:

* "Pause" — temporarily stop execution.
* "Resume" — continue execution after pausing.
* "Stop" — completely stop execution.
* "Force restart" — restart the process from the beginning.

### State tracking

In the "Process runs" section, you can view:

* A list of all process runs for the entity.
* Detailed information about each run.
* Action execution logs.
* The status of each process element.
* The process execution timeline.
* The contents of the [process store](store/#viewing-the-store-during-execution) on the "Store" tab.

#### Timeline

The run visualization panel has a "Timeline" tab that shows the process execution history: for each element that participated in the current run, the start and end times, duration, and status are shown.

#### Process log

A detailed execution log is available for each process run: a panel on the run diagram, a separate window, or a dock at the bottom of the screen. The log and the store's JSON can be downloaded from the process run dialog panel.

## Usage examples

### Creating a configured project

Typical process diagram:

1. "Start" — starts the process.
1. "Task" — creates a project in GitLab.
1. "Exclusive gateway" — checks whether creation succeeded.
1. "Task" (on success) — configures the project's variables.
1. "Task" (on failure) — sends an error notification.
1. "End" — completes the process.

### Deploying an application

Typical process diagram:

1. "Start" — starts the deployment process.
1. "Parallel gateway" — splits into branches.
1. "Task" (branch 1) — creates a namespace in Kubernetes.
1. "Task" (branch 2) — creates secrets in Vault.
1. "Parallel gateway" — waits for both branches to complete.
1. "Task" — deploys the application.
1. "End" — completes the process.

## Limitations

The following limitations apply to processes:

* Processes cannot contain more than 100 elements.
* The maximum process execution time is 24 hours.
* The number of concurrent process runs is limited by system settings.
* Some actions may not be available for use in processes.
