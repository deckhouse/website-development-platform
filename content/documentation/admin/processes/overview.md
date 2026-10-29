---
title: Overview
weight: 10
description: Creating and running processes — diagram elements, parameters, store, and execution management.
---

Processes are an automation mechanism for complex business scenarios that lets you build visual execution diagrams for actions, with support for conditional logic, parallel execution, and error handling.

## Key concepts

This section describes the main parts of a process.

### Process elements

A process consists of various types of elements:

* "Start": The process entry point.
* "Task": Runs a specific action.
* "AI agent": Sends a task to an AI agent and writes the result to the process store.
* "Exclusive gateway": Conditional branching.
* "Parallel gateway": Merges several process branches into one.
* "Loop": Repeats part of the process a fixed number of times, or iterates over a JSON array from the store.
* "Template": Evaluates a Go template and writes the result to the process store.
* "Timer": Pauses until a specified point in time.
* "Note": A text block.
* "Error": Immediately stops the process with the `Failed` status.
* "End": Completes the process.

## Creating a process

A process is created and configured in the process editor.

### Basic information

To create a process, go to "Self-service" → "Processes" and click "Create".

Fill in the following fields:

* "Name": The process name.
* "Description": A detailed description of the process's purpose.
* "Resource": One or more resources for which the process can be run.
* "Owner": The user responsible for the process.
* "Owning team": The team responsible for the process.
* "Tags": Tags used to categorize the process.
* "Icon": The icon displayed in the interface.

### Process configuration

The process is configured in the visual editor, on the "Configuration" tab.

{{< alert level="info" >}}
To configure a process, you must first fill in the basic fields and create it.
After that, the "Configuration" and "Parameters" tabs become available.
{{< /alert >}}

#### Adding elements

To add elements to the diagram:

1. Add an element by selecting its type in the editor panel.
1. Configure the element's parameters in the side panel, for example, the action and conditions.
1. Connect the elements to each other.

### Process parameters

The "Parameters" tab is used to configure process parameters that can be used in all actions within the process.

The configuration and use of process parameters are described in [Templating](../../user/templating/#process-parameters).

### Process store

The process store is a JSON object used to pass data between steps of a single run: identifiers, arrays, intermediate results, and text produced by the "Template" element.

For a full description of write rules, operations, loop context, and example scenarios, see [Process store](store/).

In brief:

- **One store per run**: Each process instance has its own store; it can be viewed on the "Store" tab when visualizing a run.
- **Nested paths**: Keys are specified with dots: `notification.module_name`, `ctx.job.id`; array indices are supported: `items[0].status`.
- **Rule-based writes**: Data is written to the store after an action completes successfully, or when the "Template" element runs.
  Available operations include setting strings and JSON, appending, merging, and deleting, plus a condition for each rule.
  Action rules are configured as described in [Process store update](../../actions/overview/#process-store-update).
- **Reading**: Go templates `{{ .store.<PATH> }}` in action configuration and gateway conditions.
  The syntax is described in the [Process store](../../user/templating/#process-store) section of the "Templating" page.

If a key is not present in the store, the template `{{ .store.<PATH> }}` will cause the step to fail with an error.

## Element types

A process diagram is built from elements of the following types.

### Task

Each task runs a specific action previously created in "Self-service" → "Actions".

### AI agent

The "AI agent" element sends a task to an [AI agent](../ai-agents/overview/), waits for the agent run to finish, and writes the result to the process store.
The agent runs on portal workers on behalf of the user who started the process.

The "Timeout (seconds)" field sets how long to wait for the agent run to finish.
If the field is empty, the wait lasts 60 minutes.
When the timeout expires, the agent run is interrupted, and the step gets the `Timeout` status.

If the agent is unavailable when the step starts, for example, a local agent does not respond, the agent run fails immediately, and the step is handled as an [agent error](#agent-error).
To run the step again, restart the process.

The element configuration includes the following fields:

| Name | Required | Description |
| --- | --- | --- |
| "Executor" | Yes | "Agent" from the catalog or "Workspace" |
| "Workspace" | For the "Workspace" executor | Workspace UUID or a Go template, for example, `{{ .process.workspace_uuid }}` |
| "Agent" | For the "Agent" executor | Agent identifier. You can specify a Go template, for example, `{{ .process.agent_slug }}` |
| "Agent message" | Yes | Go template of the message that the agent receives. The `.process`, `.store`, and `.context` roots are available |
| "Output store key" | Yes | Process store dot-path where the result is written, for example, `agent_result` or `analysis.plan`. Defaults to `agent_result` |
| "AI provider" | For agents whose model is provided by the portal | AI provider name. Required for the platform agent and for external agents with the DDP model proxy allowed |

The AI provider is looked up by name among the providers of the user who started the process.
A provider from a template with a shared token is available to everyone allowed to use the template, and a personal provider is available only to its owner.
If the provider is not set in the element, it can be selected when starting the process.
Without a provider, the process does not start and returns an error.

The agent gets the tools of its assigned MCP collections that the process initiator can use.
Without collections, the agent works without MCP tools.
Running the step requires the `use:ai-agents` permission or `use:ai-agent` on the selected agent.

In the "Executor" field, you can also select "Workspace".
In this mode, the step sends the job to the attached chat of the initiator's workspace bound to the process entity.

#### Agent result

After the agent run finishes, the portal writes the following object to the store under the key from the "Output store key" field:

```json
{
  "type": "agent_run",
  "runUuid": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "agentSlug": "ddp-agent",
  "status": "succeeded",
  "answer": "Agent response text",
  "finishedAt": "2026-10-09T10:15:00Z"
}
```

The `status` field takes the `succeeded`, `failed`, or `cancelled` value; on error, the `error` field with the error text is added.
If the key already exists in the store, its value is overwritten.

Subsequent elements get the agent response through the store, for example, `{{ .store.agent_result.answer }}`.
A key with a dot is written as a nested object.
For example, the result under the `analysis.plan` key is read as `{{ .store.analysis.plan.answer }}`.

#### Agent error

If the agent run fails, the behavior depends on the element's connections.
If a connection to an exclusive gateway or a conditional connection leaves the element, the portal writes the result with the `status` and `error` fields to the store and continues the process along the outgoing connections.
To route the process to an error-handling branch, check the `{{ eq .store.agent_result.status "succeeded" }}` condition in the gateway.
Otherwise, the process branch ends at this element, and the error is reflected in the final process status.

If the user stops the agent run, the process branch ends at this element without following the outgoing connections, and the cancellation is reflected in the [final process status](#process-statuses).

If the agent run cannot be created, for example, the "Agent message" template refers to a key missing from the store, the step also ends with the `Failed` status.
The error reason is shown on the "Execution log" tab of the step side panel.

If the agent run succeeds but the result cannot be written to the path from the "Output store key" field, the step ends with the `Failed` status, and the process does not continue along the outgoing connections.
Writing is impossible when the path goes through a value that is not an object, for example, the `catalog.status.answer` key when `catalog.status` contains a string.

#### Repeated execution

Each time the process passes through the element again, for example, on a new loop iteration, the portal creates a new agent run.
Agent runs appear in the AI agent [run history](../ai-agents/runs/) with the "Process" trigger, and the agent response and tool calls appear in the element side panel on the run diagram.

### Exclusive gateway

An exclusive gateway lets you configure process branching based on conditions.
By default, the status of the previous task is checked, and depending on it, execution follows the "success" or "failure" branch.

As conditions for an exclusive gateway, you can set either a check of the previous task's status, or a comparison of values from templates:

- To check the status of the previous task, use the construct `{{ .prev_task.status }}`.
- To check a value from the store, use `{{ .store.<PATH> }}`, including nested keys: `{{ .store.notification.status }}`.
- Inside a loop: Fields of the current item: `{{ .store._loop.item.scan_type }}`, and the flags `{{ .store._loop.first }}` and `{{ .store._loop.last }}`.

You can check multiple conditions and combine their results using the `AND` or `OR` operator.

If the condition check passes, the process follows the "True" branch (green); if not, it follows the "False" branch (red).
The "True" and "False" ports can be placed on any side of the gateway; a connection can be moved to a different port.

### Parallel gateway

Used to merge several branches into one, based on specified conditions.

In the parallel gateway's configuration, you can choose whether to wait for all incoming elements or for at least one incoming element to complete before continuing execution.

You can also configure what exactly counts as "completion": only successful completion of the tasks feeding into the gateway
or any final status of the incoming tasks (for example, `Failed` or `Skipped`).

A parallel gateway can also be used to split branches, if connected after "Exclusive gateway" or "Loop" elements, or as a helper element to improve the readability of the process diagram.

#### Merging branches in a loop

If a parallel gateway merges branches inside a loop or on a return path through a back connection, on each pass the gateway counts only the branches that finished during this pass.
A branch finished on a previous iteration is not counted.

Therefore, each branch entering the gateway must reach it on every pass.
If a branch does not reach the gateway within the element timeout (60 minutes by default), the gateway fails with the `ParallelGateway Join timed out` error and the list of unfinished branches.
The timeout is counted from the start of the current pass.

The `tasksCompleted` and `tasksCompletedAny` conditions of the exclusive gateway check whether a task has finished at least once during the whole process run, not only during the current pass.

### Loop

The "Loop" element repeats a branch of the process either a fixed number of times, or once for each element of a JSON array from the store.
Once all iterations are complete, the element connected to the loop's exit is activated.

#### Loop mode

Two modes are available.

In the "Fixed count" mode, set the "Number of iterations" from 1 to 10000.
On each iteration, `{{ .store._loop.item }}` holds the iteration number (1, 2, 3…).

In the "Foreach collection" mode, set the "Collection template": a Go template that, when entering the loop, must produce a JSON array.
Typically, this is a path to an array written by a previous task: `{{ .store.notification.engagements }}`.
Each array element is one iteration; the current element is available as `{{ .store._loop.item }}`.
The array can contain up to 10,000 items; a larger array fails the loop with an error.

If the array is empty or the key is missing, the loop body is not executed, and the process immediately follows the "Exit" connection.

For more information, see [Process store](store/#looping-over-a-collection-from-the-store).

#### Outgoing connections

Exactly two connections to different elements must originate from the "Loop" element:

1. "Loop body": The branch built from this port is executed on each iteration.
1. "Exit": Execution continues along this branch after the last iteration.

The "Loop body" and "Exit" ports can be placed on any side of the element; a connection can be moved to a different port.

#### Usage

Typical scenarios:

- Repeatedly checking the status of an external system with a fixed number of attempts.
- Iterating over a list of objects from an API response: a task writes the array to the store, and a loop in "Foreach collection" mode processes each element.
- Nested loops — the parent context is available via `{{ .store._loop.parent }}`.

### Template

The "Template" element evaluates a Go template without calling an additional action, and writes the result to the store as a string.

Configure:

- "Template body": Text with placeholders `{{ .store.* }}`, `{{ .process.* }}`, and, inside a loop, `{{ .store._loop.* }}`.
- "Output store key": The destination dot-path, for example, `rendered_message`.
- "Format hint": How to display the value when viewing the run (`text`, `markdown`, `html`, `json`); does not affect the store's content.

For examples and limitations, see [Process store](store/#the-template-element).

### Timer

The "Timer" element pauses process execution until a specified point in time, after which the next element is activated.

While the process is only waiting for the timer to fire (no other active tasks), the run is placed in the `Wait` status.
The run visualization panel shows the banner "Waiting on timer — not before {ends} ({remaining})" with the estimated resumption time.

#### Outgoing connections

Exactly one connection to the next element must originate from the "Timer" element.
Connection is only possible through the left (input) and right (output) ports.

If the timer is passed through again within the same run (for example, via a loop), the trigger time is recalculated.

#### Schedule modes

In the timer's configuration, select a "Schedule".
The "Delay after this element starts" option sets a fixed pause from the moment the process reaches the timer.
Set the "Delay (seconds)" from 1 to 1,209,600 (14 days).
The "Custom schedule" option triggers the timer at a specified calendar time in the selected time zone (IANA format, for example, `Europe/Moscow`).

For "Custom schedule" mode, specify a "Pattern":

- "Specific weekday": "Day of week", "Time of day" (hour and minute), "Time zone".
- "Specific day each month": "Day of month" (1–31), "Time of day", "Time zone".
  If the month does not have that day, the last day of the month is used.
- "Every N days": "Every N days" (1–365), "Time of day", "Time zone".
  The first trigger is no earlier than N calendar days from the day the process reached the timer; upon re-entering the element, the calculation is redone.

"Time of day" is set in the selected time zone (the "Time zone" field), if it differs from the browser's time zone.

#### Usage

Typical scenarios: a pause before re-checking the status of an external system, a delayed start of the next step, waiting for a maintenance window, or a regular calendar slot.

### Note

The "Note" element is intended for text annotations on the process diagram: it is not executed at runtime and is not connected to other elements.

In the note's configuration, you can set:

- "Text": Content with Markdown support.
- "Background color": The fill color.
- "Text color": The text color.

A note is always positioned behind other elements and can be used to visually highlight parts of the process.
It is also the only element whose size can be changed.

### Error

The "Error" element forcibly transitions a running process to the `Failed` status: when this element is reached, execution of all process actions is interrupted.

No connections to subsequent elements should originate from this element.

## Running a process

A process can be run manually or by an automation.

### Manual run

To run a process manually:

1. Go to the entity for which the process needs to be run.
1. In the entity's menu, select "Run process".
1. Choose the desired process from the list.
   If only one process is available, it is selected automatically and the list is not shown.
1. Fill in the process parameters.
1. Click "Run".

After the run starts, the notification "Process "{name}" started" is shown.

### Launch parameters

The "Run process" dialog opens in simplified mode.
It shows only the process parameters with the ["Show in simplified form"](../../user/properties/#configuration) flag and the required parameters that have no value.
The other parameters keep their default values and are passed at launch.

Above the parameters, the hint "Showing important parameters: {important} of {total}" is shown.
If there are no important parameters, "No important parameters" is shown.
If nothing needs to be filled in, the message "This process has no parameters to fill in — click "Run" to start it" is shown.

The "All process parameters" button switches the dialog to full mode, and the "Important parameters only" button switches it back.
Full mode has the following sections:

* "Common process properties": Parameters defined in the process configuration.
* "Process team properties": The team whose variables are used during execution.
* "Configuration": The process diagram. Click a task to enable or disable its action and fill in its parameters.

The process description is shown under the title in a shortened form; the "more" link expands it.

If a required process parameter is not filled in, clicking "Run" shows the error "Fill all required process properties".
If a required field of an enabled task is not filled in, the dialog switches to full mode and shows the error "Fill all required fields for all enabled tasks".

### Launch context

The launch context is a set of values that a portal section passes to the process at launch, for example, the environment and the system selected on the Home page.
The user does not enter the context, and it is not stored in the process configuration.

In process templates, the context is available via the `.context` root, for example, `{{ .context.environment.slug }}`.
A process run from the catalog receives an empty context.

## Execution management

While a process runs, you can track its status and control execution.

### Process statuses

The run status is shown as one of the following values:

* `Running`: The process is running.
* `Wait`: The process is only waiting for a timer to fire.
* `Unapproved`: All active tasks are waiting for action approval.
* `Paused`: Process execution is paused.
* `Resuming`: The run is being restored after a portal restart.
* `Completed`: The process reached an "End" element, and no task failed or was declined.
* `Partially Completed`: The process reached at least one "End" element, but some tasks failed or were declined.
* `Failed`: The process finished with an error: no "End" element was reached, or the process reached an "Error" element.
* `Canceled`: The run was stopped or canceled by a user.

### Management operations

Run management buttons are shown on the run card in the "My Activities" panel, on the "Processes" tab.
The set of buttons depends on the run status:

* "Retry process": Restarts the process from the "Start" element; the run timeline is preserved.
  A run in the `Running`, `Wait`, `Paused`, `Resuming` or `Unapproved` status cannot be retried: stop it first.
* "Pause process": Pauses a run in the `Running` status.
* "Resume process": Resumes a run in the `Paused` status.
* "Cancel process": Cancels a run in the `Running` or `Paused` status. The run gets the `Canceled` status.
* "Stop process": Stops a run in another non-final status, for example, `Wait` or `Unapproved`.
  The run gets the `Canceled` status.
* "Delete process": Deletes the run.

The same operations are available in the "Run history" table on the "Processes" tab of the entity.
There, canceling and stopping are performed by one button.

### State tracking

While the user has running processes, the "My Activities" panel icon in the top bar of the portal is replaced by an indicator.
The "Processes" tab of the panel shows the number of running processes and the line "Processes running: {count}"; the list is updated when a process starts and finishes.

In the "Run history" table on the "Processes" tab of the entity, you can view:

* A list of all process runs for the entity.
* Detailed information about each run.
* Action execution logs.
* The status of each process element.
* The process execution timeline.
* The contents of the [process store](store/#viewing-the-store-during-execution) on the "Store" tab.

#### Timeline

The run visualization panel has a "Timeline" tab that shows the process execution history: for each element that participated in the current run, the start and end times, duration, and status are shown.

#### Run diagram

While a run is in progress, the diagram is updated every 5 seconds.
The element state is shown by a mark in the corner of the element: completed, completed after retry, error, declined, cancelled, running, waiting, skipped, or awaiting approval.
For a finished run, the side panel of an element that did not finish by itself shows the final run status and its completion time.
A task awaiting approval can be approved with the "Approve" button directly on the diagram.
The button is active only for users from the action's list of [approvers](../../actions/overview/#approval).

The traversed path is highlighted, and connections the process did not follow are shown in light gray.
Elements traversed on previous loop iterations are dimmed, and the loop shows "Iteration {current} of {total}".
If an element ran several times, its side panel shows the log of each pass separately with the "Visit {n}" heading.

#### Process log

A detailed execution log is available for each process run: a panel on the run diagram, a separate window, or a dock at the bottom of the screen.
The log and the store's JSON can be downloaded from the process run dialog panel.

## Usage examples

The following examples show typical processes.

### Creating a configured project

Typical process diagram:

1. "Start": Starts the process.
1. "Task": Creates a project in GitLab.
1. "Exclusive gateway": Checks whether creation succeeded.
1. "Task" (on success) — configures the project's variables.
1. "Task" (on failure) — sends an error notification.
1. "End": Completes the process.

### Deploying an application

Typical process diagram:

1. "Start": Starts the deployment process.
1. "Parallel gateway": Splits into branches.
1. "Task" (branch 1) — creates a namespace in Kubernetes.
1. "Task" (branch 2) — creates secrets in Vault.
1. "Parallel gateway": Waits for both branches to complete.
1. "Task": Deploys the application.
1. "End": Completes the process.

## Limitations

The following limitations apply to processes:

* A "Loop" element in "Fixed count" mode runs from 1 to 10,000 iterations.
* The collection of a "Loop" element in "Foreach collection" mode can contain up to 10,000 items.
  A larger collection fails the loop with an error.
* The "Delay (seconds)" field of a "Timer" element accepts values from 1 to 1,209,600 (14 days), and the "Every N days" field accepts values from 1 to 365.
* The body of a "Template" element is limited to 1 MiB.
* If the "Timeout (seconds)" field of a task is greater than zero, the task is interrupted after this time and gets the `Timeout` status.
