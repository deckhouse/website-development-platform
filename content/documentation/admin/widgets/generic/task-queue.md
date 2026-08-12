---
title: Task queue
description: Monitor background task queues, active workers, and task processing status.
weight: 150
---

The widget monitors the task queue and the workers that process tasks in the background. It displays queue statistics, worker (consumer) information, and details of every task in the queue.

## Displayed data

The widget has three main sections.

### Queue statistics

The top of the widget displays four key metrics:

- **Queue size** — Total number of tasks in the queue.
- **Pending tasks** — Number of tasks waiting to be processed.
- **Active workers** — Number of active workers (consumers) processing tasks.
- **Queued tasks** — Total number of new and in-progress tasks.

### Workers table

The table provides information about each active worker:

- **Consumer name** — Worker (consumer) identifier.
- **Pending tasks** — Number of tasks assigned to the worker and waiting to be processed.
- **Idle time** — Time since the worker's last activity.

{{< alert level="info" >}}
The table displays only active workers. Workers that are not processing tasks and have been inactive for more than five minutes are automatically hidden.
{{< /alert >}}

### Tasks table

The table provides details about every task in the queue:

- **Task UUID** — Unique task identifier.
- **Type** — Task type, for example, `health_check`.
- **Resource UUID** — Identifier of the resource or entity associated with the task.
- **Consumer** — Name of the consumer processing the task.
- **Idle time** — Time since the task was delivered to the worker.
- **Delivery time** — Time when the task was delivered to the worker.
- **Status** — Current task status:
  - **New** — The task has been added to the queue but has not yet been assigned to a worker.
  - **In progress** — The task has been assigned to a worker and is being processed.

## Configuration

The widget requires no additional configuration and works immediately after you add it to the dashboard.
