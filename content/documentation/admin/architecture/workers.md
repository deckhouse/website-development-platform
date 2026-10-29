---
title: Workers
description: Configuring, monitoring, and scaling the workers that run Deckhouse Development Portal background tasks.
weight: 20
---

Workers are Deckhouse Development Portal (DDP, portal) components that process tasks from a queue in the background.
They run various operations asynchronously, such as:

- Synchronizing data from datasources
- Running actions and processes
- Running entity status checks
- Running AI agents from "AI agent" process elements
- Periodically synchronizing catalogs of external MCP servers

Workers run independently of the main application and let you scale the system load by moving resource-intensive operations to dedicated replicas.

## Configuration

Worker parameters are set in the `spec.settings.workers` section of the `development-platform` ModuleConfig or in the module settings in the Deckhouse web UI.
The following parameters are available.

### Number of replicas

The `workers.replicas` parameter sets the number of workers running at the same time.
Default value: `2`.
Each worker processes tasks independently of the others.

### Maximum number of parallel tasks

The `workers.maxTasks` parameter sets how many tasks each worker processes in parallel. Default value: `10`.

{{< alert level="info" >}}
The total number of tasks processed simultaneously by the whole system is calculated as `workers.replicas × workers.maxTasks`.
For example, with 2 workers and 10 tasks per worker, the system can process up to 20 tasks at the same time.
{{< /alert >}}

## Monitoring

To monitor workers and the task queue, use the ["Task queue"](../../widgets/generic/task-queue/) widget, which shows:

- Queue size (total number of tasks)
- Number of pending tasks
- Number of active workers (consumers)
- Detailed information about tasks in the queue
- Status of each task (new, in progress)

## Types of processed tasks

Workers process the following types of tasks:

### Entity status check tasks

Status check tasks are run by workers, which determine the state of entities based on the configured rules.
This approach lets you:

- Reduce the load on the main server
- Keep the system stable with a large number of entities
- Guarantee that checks run even when the main application is under high load

### Datasource synchronization tasks

Workers process tasks that synchronize data from external sources, which lets you:

- Run synchronization in the background
- Avoid blocking the user interface
- Process large amounts of data

### Action execution tasks

Long-running actions are processed by workers, which provides:

- Asynchronous execution of actions
- The ability to track execution progress
- A stable user interface

### AI agent run tasks

Workers run [AI agents](../ai-agents/overview/) from "AI agent" process elements: they send the task to the agent over the A2A protocol and wait for it to complete.
A single run is limited to 70 minutes.
While a run is in progress, it occupies one of the worker's `workers.maxTasks` parallel tasks.

### MCP server synchronization

Periodic synchronization of MCP server catalogs is performed by workers.
Enable it in the server settings and specify a system account.
If an attempt fails, the previous catalog is kept; the results and update time are shown on the server card.
For setup instructions, see the [MCP guide](../../user/mcp-management/#mcp-servers).

## Background ClickHouse data operations

If [ClickHouse](../../install/#clickhouse) is connected, workers also move data to it from PostgreSQL and clean up obsolete data.
For example, this is how the history of entity property changes is maintained:

- Moving new change records from PostgreSQL to ClickHouse
- Rebuilding queue indexes in PostgreSQL after a large volume of changes is moved
- Deleting the history of entities and properties that no longer exist
- Deleting history that is past its retention period

These operations are not taken from the task queue.
Each worker starts them on its own, and a distributed lock prevents them from running in several replicas at the same time.
If no worker is running, property changes are still written to PostgreSQL and remain visible in the interface, but the transfer queue is not processed and keeps growing.

Transfer parameters are set in the `clickhouse.replication` section of the module settings: the transfer batch size and interval, and index rebuilding.
The history retention period and cleanup interval are set in the `clickhouse.propertyHistory` section.

## Scaling

To increase system performance, you can:

1. Increase the number of worker replicas to process more tasks at the same time.
1. Increase the number of parallel tasks per worker to improve the utilization of each worker.

{{< alert level="info" >}}
When increasing the load on workers, make sure the cluster has enough resources (CPU, memory) to process all tasks.
{{< /alert >}}

## Troubleshooting

This section describes common problems with workers and how to resolve them.

### Workers do not process tasks

If tasks accumulate in the queue and are not processed:

1. Check that the workers are deployed and running (using the "Task queue" widget).
1. Make sure the number of worker replicas is greater than 0.
1. Check the worker logs for errors.
1. Make sure the workers have enough resources (CPU, memory).

### Slow task processing

If tasks are processed slowly:

1. Increase the number of worker replicas.
1. Increase the number of parallel tasks per worker (if resources allow).
1. Check the performance of the external systems the workers interact with.
1. Make sure there are no bottlenecks in the network or database.
