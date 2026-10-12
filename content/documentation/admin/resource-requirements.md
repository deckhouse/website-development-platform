---
title: Compute resource requirements
description: Demo installation of Deckhouse Development Portal in Deckhouse Platform and CPU, memory and disk sizing for 10, 100, 200, 500 and 1000 users.
weight: 12
---

Deckhouse Development Portal (DDP) is installed by the `development-platform` module in a Deckhouse Platform (DP) cluster.

Allocate the following CPU, memory and disk for the installation scenarios:

| Profile | DDP: CPU / memory | PostgreSQL: CPU / memory / disk | Redis: CPU / memory / disk | ClickHouse: CPU / memory / disk |
| --- | --- | --- | --- | --- |
| Demo installation | 4 CPU / 8 GB, including managed database instances | CPU and memory included in the demo installation. Disk 8 GB | CPU and memory included in the demo installation. Disk 8 GB | CPU and memory included in the demo installation. Disk 16 GB |
| 10 users | 8 CPU / 32 GB | 2 CPU / 8 GB / 16 GB | 2 CPU / 4 GB / 8 GB | 4 CPU / 16 GB / 32 GB |
| 100 users | 16 CPU / 32 GB | 2 CPU / 16 GB / 64 GB | 2 CPU / 8 GB / 16 GB | 4 CPU / 16 GB / 64 GB |
| 200 users | 16 CPU / 64 GB | 2 CPU / 16 GB / 64 GB | 2 CPU / 16 GB / 32 GB | 4 CPU / 16 GB / 128 GB |
| 500 users | 32 CPU / 64 GB | 4 CPU / 64 GB / 256 GB | 2 CPU / 32 GB / 64 GB | 4 CPU / 16 GB / 256 GB |
| 1000 users | 32 CPU / 128 GB | 16 CPU / 64 GB / 512 GB | 2 CPU / 32 GB / 128 GB | 4 CPU / 32 GB / 512 GB |

A budget is the sum of the component resources rounded up to the nearest power of two.
The table lists budgets.

For the demo installation, the volume sizes are taken from the configuration example below.
In the demo installation, the DDP budget includes the API, the web interface, the platform AI agent, one worker
and the managed PostgreSQL, Valkey and ClickHouse instances.
In the other profiles, the resources of the portal and of each external database are listed separately.
The databases in these profiles are connected in the `external` mode.

The budget for each database is given per server.
It is sized according to the scenarios and assumptions below.
CPU is sized by the rate and cost of operations, memory by the working set and concurrency,
disk by the accumulated data and the space for maintenance.
Additionally, account for the resources of all replicas, backups and service components.

The sizing covers the first 180 days of operation.
How the values are obtained is described in [the database CPU and memory sizing](#database-cpu-and-memory-sizing)
and [the disk space sizing](#disk-space-sizing).

The portal budget is the free resources for DDP pods.
Additionally, account for the consumption of DP, the operating system, other applications,
temporary pods during updates and the margin for node failures.

DP version 1.72 or newer with the `development-platform` module available,
the [user-authn](/modules/user-authn/) module and persistent storage for the databases in the cluster are required.
The installation steps and the database connection are described in the [installation guide](install/).

## Demo installation

The demo installation scenario assumes one user, a small catalog and one or two processes running at the same time.
For the demo installation, use PostgreSQL, Redis and ClickHouse in the `managed` mode.
Enable the DP modules [`managed-postgres`](/modules/managed-postgres/configuration.html#enable), [`managed-valkey`](/modules/managed-valkey/configuration.html#enable) and [`managed-clickhouse`](/modules/managed-clickhouse/configuration.html#enable) before the installation.
In this mode, Redis is replaced with the compatible Valkey.
ClickHouse keeps the property history, which is available to demonstrate the portal capabilities.

For DDP together with the managed database instances, allocate the following resources in a running DP cluster:

| Parameter | Value |
| --- | --- |
| User load | 1 user |
| CPU | 4 CPU |
| Memory | 8 GB |
| Processes running at the same time | 1–2 |
| Persistent disk | 32 GB for PostgreSQL, Valkey and ClickHouse |
| `workers.replicas` | `1` |
| `workers.maxTasks` | `5` |
| ClickHouse | Enabled, `managed` mode |

With two processes that have two active steps each, the worker runs four process tasks
and one additional background task at the same time.
The resources of DP itself, container images and temporary disk are added to this budget.

Add the following settings to the ModuleConfig `development-platform`. Keep the other installation parameters unchanged:

```yaml
spec:
  settings:
    postgres:
      mode: managed
      instance:
        persistentVolumeClaim:
          size: 8G
    redis:
      mode: managed
      instance:
        persistentVolumeClaim:
          size: 8G
    clickhouse:
      mode: managed
      instance:
        persistentVolumeClaim:
          size: 16G
    workers:
      replicas: 1
      maxTasks: 5
```

Set the CPU and memory of the managed instances in `instance.cpu` and `instance.memory`
and include them in the total demo installation budget together with the portal components.
Set the volume sizes in `instance.persistentVolumeClaim.size`.
The `G` suffix in the example sets the size in decimal GB.
The class of the managed instance must allow the selected CPU, memory and volume sizes.
The instance parameters are described in the [installation guide for managed instances](install/#installation-with-managed-instances).

When the catalog, the number of tasks running at the same time or the number of active users grows,
switch to the profiles with user load.

## Load scenarios

Select a profile by the expected number of active users and the catalog size.
In the examples, one running process has two steps active at the same time,
each of which takes one worker slot.
Background tasks include data source synchronizations, status checks and other operations from the shared queue:

| Scenario parameter | 10 users | 100 users | 200 users | 500 users | 1000 users |
| --- | --- | --- | --- | --- | --- |
| Active users (working at the same time) | 2 | 20 | 40 | 100 | 200 |
| Processes running at the same time | 2 | 5 | 10 | 15 | 20 |
| Worker tasks from processes | 4 | 10 | 20 | 30 | 40 |
| Background tasks running at the same time | 6 | 10 | 20 | 30 | 40 |
| Total tasks running at the same time | 10 | 20 | 40 | 60 | 80 |
| Entities in the catalog | 1000 | 10,000 | 20,000 | 50,000 | 100,000 |
| Catalog resources that produce events | 10 | 20 | 40 | 80 | 100 |
| `workers.replicas` | `1` | `2` | `4` | `6` | `8` |
| `workers.maxTasks` | `10` | `10` | `10` | `10` | `10` |

Select a profile by all scenario parameters together.
The number of registered users affects the accumulation of process runs,
and the number of active users affects the request rate to the API and the databases.
An installation for 10 users with frequent synchronizations of a large catalog may need more workers than the 100-user profile.
The number of entities alone does not define the required memory either: consider the size of the parameters and the volume of one synchronization.

The API, the web interface and the platform AI agent run with one replica in every profile.
The number of workers increases the concurrency of background tasks.
For the 1000-user profile, check the performance of the API and the databases separately.
Increasing the number of workers does not increase the performance of all components at once.

## Sizing assumptions

The profiles are sized for the following data volume and activity.
If these parameters change, recalculate the corresponding part of the budget:

| Parameter | Value in the sizing |
| --- | --- |
| Data reads | 1 batch of requests per second per active user at peak. 90% of batches read the catalog and events, 10% search the whole catalog |
| Property history views | 10% of active users at the same time. Up to 1 request per second per such user |
| Process runs | 2 runs per user per working day |
| Run accumulation horizon | 180 calendar days, approximately 129 working days |
| Data size of one run | 256 KiB in total for the process, its actions, context, results and logs |
| Size of a current entity | 2 KiB |
| Entity changes | 5 changes per day, each creates an event and a property history record |
| Size of a full event | 8 KiB, including the old and new state of the entity |
| Event retention in PostgreSQL | 4 days plus up to 1 day waiting for the next cleanup |
| Redis event streams | About 10,000 events per catalog resource. New events extend the lifetime of the stream |
| Synchronization peak | 100 times the average daily change rate |
| Size of a property history record | 1 KiB before compression |
| Property history retention | 180 days |
| Task queue | Up to 10,000 tasks of 16 KiB each |
| Temporary task result | 64 KiB, kept for 3 minutes |
| Task duration for the exchange sizing | 5 seconds. The number of completions per second equals the number of worker slots divided by 5 |
| Event-driven automations | Up to 2 recipients per event. The tasks they create count towards the shared concurrency limit |
| Free disk | 30% after placing the data and the maintenance margin |

The record sizes are profile parameters and include the payload.
Record sizes use KiB: 1 KiB equals 1024 bytes.
The sizing covers indexes, service structures, caches and temporary copies with the additional factors below.
Process and action records keep accumulating after the selected horizon.
Plan further disk expansion.

## DDP component resources

For the 10–1000-user profiles, the budget is made up of the following component resources.
The table lists approximate values per instance before rounding:

| Component | CPU | Memory |
| --- | --- | --- |
| API (`backend`) | 4 CPU | 8.59 GB |
| Web interface (`frontend`) | 1 CPU | 2.15 GB |
| Platform AI agent (`agent`) | 1 CPU | 1.07 GB |
| One worker (`worker`) | 2 CPU | 8.59 GB |

For `N` workers, the sum of the resources of the API, the web interface, the AI agent and the workers is
`6 + 2 × N` CPU and about `11.81 + 8.59 × N` GB of memory.
For example, for two workers the sum for the portal is `6 + 2 × 2 = 10` CPU
and approximately `11.81 + 8.59 × 2 = 28.99` GB of memory.
After rounding up, the budget is 16 CPU and 32 GB of memory without the databases.

## Database CPU and memory sizing

The CPU sizing is based on the peak operation rate and their cost in CPU time.
The sizing includes a twofold margin for CPU differences and a target load of up to 50%.
Therefore, the consumption `Σ(operations per second × ms of CPU per operation) / 1000`
is multiplied by 4 and then rounded up to a power of two.

The cost of typical operations is taken from measurements of queries and commands with synthetic data.
For history queries, the history of one entity with 900 records over 180 days was used:

| Operation | Operation cost in the sizing |
| --- | --- |
| PostgreSQL: batch of catalog and event reads | 2 ms. For related object queries and access checks, 4 ms |
| PostgreSQL: search across 100,000 entities | 40 ms. Scales proportionally to the catalog size |
| PostgreSQL: entity update with history and event | 1 ms, with a margin for an 8 KiB event |
| Redis: write or delivery of an 8 KiB event | 0.03 ms |
| Redis: short coordination command | 0.01 ms |
| Redis: task and result exchange between a worker and the backend | 0.4 ms, with a margin for separate commands and additional requests |
| ClickHouse: query of a page, a history window or the state on a date | 16 ms |
| ClickHouse: insert of a record with a margin for merges | 0.004 ms per record |

Large responses, other kinds of search and more frequent requests change the cost of operations.
Use SSD for the disks. Read and write latency also affects the response time.

### PostgreSQL CPU and memory

The CPU sizing accounts for user reads, search and the peak change flow from synchronizations.
The minimum budget in the model is 2 CPU.
For example, in the 1000-user profile there are 200 active users
and up to 579 entity changes per second arrive at peak.
The estimated consumption is about 2.1 CPU.
After multiplying by 4 and rounding, the budget is 16 CPU.

The memory working set includes the current catalog, 5 days of events
and the run data for the last 30 calendar days, approximately 22 working days.
The working set volume is multiplied by 1.5 for indexes and caching.
The sizing additionally adds a service margin of 2 GB,
32 MiB for the reads of each active user
and 8 MiB for each of the 10 parallel updates of one background task.
The minimum memory budget in the model is 8 GB.
Account for parallel operations and connections in the PostgreSQL settings as well.
PostgreSQL consumes the memory for sorts and hash operations per operation, as described
in the [PostgreSQL documentation](https://www.postgresql.org/docs/current/runtime-config-resource.html).

### Redis CPU and memory

The CPU sizing includes the following Redis operations:

- Publishing events.
- Reading events and acknowledging them by automations.
- Enqueueing tasks.
- Reading new and pending tasks.
- Acknowledging and deleting completed tasks.
- Setting and renewing locks.
- Publishing and receiving results.

Each event includes a write, a retention extension and a read with acknowledgement by two recipients.
The sizing additionally accounts for up to 35 short commands per second per worker
and up to 2 requests per second per active user.
In the 1000-user profile, this is about 4500 commands per second.
The minimum budget in the model is 2 CPU, including a margin for persistence.
For the given rates, the CPU sizing stays within this budget.

For memory, the sizing accounts for full streams: `number of catalog resources × 10,000 events`.
The limit applies to each resource stream separately. Redis maintains it approximately.
A new event extends the lifetime of the whole stream.
The event retention in PostgreSQL does not limit the age of events in an active Redis stream.
If you change `events.streamMaxLen`, recalculate the stream volume.
With a value of `0` or less, the length limit is disabled.

The sizing adds a queue of 10,000 tasks and the results of tasks completed in the last 3 minutes to the events.
The total volume is multiplied by 1.3 for Redis structures
and by another 2 for the temporary copy during persistence.
The sizing adds 0.5 GB of service margin and 1 MiB of buffers per active user.
The minimum memory budget in the model is 4 GB.

In the 1000-user profile, the payload takes about 8.54 GB.
The sizing with margins gives about 23 GB and is rounded up to 32 GB.
The effect of persistence is described in the [Redis documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/).

### ClickHouse CPU and memory

The CPU sizing accounts for history queries, restoring properties on a date,
inserting changes and background merges.
With 20 history queries per second and a peak of 579 inserts per second,
the estimated consumption is about 0.32 CPU.
After multiplying by 4, the consumption stays below the minimum budget of the model, 4 CPU.
Therefore, the CPU is the same in the listed profiles.

The memory sizing is made up of 8 GB of service margin,
0.5 GB per user who views the history at the same time
and a cache of 25% of the history volume for the last month.
The sizing additionally accounts for four copies of a batch of 10,000 records during insert processing.
The minimum memory budget in the model is 16 GB.
For the 1000-user profile, the sizing gives about 22 GB and is rounded up to 32 GB.
The memory consumption of reads and merges depends on the query, not only on the number of returned rows.
The dependency of resources on queries is described
in the [ClickHouse recommendations](https://clickhouse.com/docs/guides/oss/best-practices/sizing-and-hardware-recommendations).

For several database servers, multiply the budget by their number.
If a server serves other applications, add the resources for DDP to the budgets of those applications.

## What drives consumption

The main sources of load differ by component.
Before selecting a profile, collect the following parameters:

| Load source | What to consider | Main consumer |
| --- | --- | --- |
| User activity | Number of active users, request rate, catalog search and filtering, number of open widgets | API and PostgreSQL. Widgets also call external systems |
| Process execution | Number of processes running at the same time, parallel branches, action duration, size of the context and results | API keeps the execution state. Workers run the tasks. PostgreSQL stores the run records |
| Catalog synchronization | Number of data sources, schedule, number and size of objects in one fetch, entity matching and relation creation | Worker memory and CPU, PostgreSQL, external APIs |
| Status checks | Number of entities and rules, frequency, check duration | Workers and external systems |
| Property history | Number of changes, size of the old and new value, retention period, queries over long periods | PostgreSQL before the transfer, ClickHouse after the transfer, background worker operations |
| Events and task exchange | Rate and size of events, number of streams, queue length, locks, size of results and number of task completions | Redis: CPU, memory and disk. PostgreSQL stores the events |
| Execution logs | Volume of run records and log detail | PostgreSQL and disk |
| Templates and repositories | Size of packs and repositories, number of concurrent operations | Worker CPU, memory and temporary disk |
| AI | Number of parallel runs, dialog length, size of tool responses, provider response time | API, AI agent, workers and PostgreSQL |

During synchronization, DDP processes the collection of fetched objects and loads the entities of the corresponding catalog resource.
A large fetch increases memory consumption even with a small number of users.
Reducing the number of parallel synchronizations reduces their concurrent memory consumption.

Account separately for the resources of the AI model service and GPU when you host the model yourself.
For workspaces, separately sum up the CPU, memory and storage requests from the templates of all workspaces running at the same time.
Add the resources of the preparation tasks.
The profile budgets include the permanent portal components.
Workspaces, development environments, CI jobs and applications that are created through DDP are added to the profile budgets.

## Worker count sizing

One worker processes up to `workers.maxTasks` tasks from the queue at the same time.
The total concurrency limit is `workers.replicas × workers.maxTasks`.
The profiles use `workers.maxTasks: 10`, so for 10, 20, 40, 60 and 80 tasks
the profiles use 1, 2, 4, 6 and 8 workers.

The number of running processes and the number of occupied worker slots differ.
One process with parallel branches can occupy several slots.
A process does not occupy a worker slot while it waits for user confirmation.
Also include synchronizations, status checks, MCP calls and AI agent runs in the total sizing.
An AI agent run started from a process occupies a slot until the task completes.
The transfer of the history to ClickHouse and its cleanup run in addition to the queue tasks
and also consume worker resources.

For a constant task flow, estimate the average number of occupied slots as the sum of `λ × t` over all task types.
Here `λ` is the number of incoming tasks per second and `t` is the average task duration in seconds.
For example, 2 tasks per second with a duration of 5 seconds occupy 10 slots on average.
With `workers.maxTasks: 10`, such a flow fully loads one worker.
Add a concurrency margin for peaks and long tasks.
The concurrency limits of the profiles do not define the throughput in tasks per second.

Increasing `workers.maxTasks` increases the concurrent memory consumption of one pod
and the number of requests to PostgreSQL and external systems.
For example, one synchronization can update the parameters of up to 10 entities in parallel.
With ten such tasks for different catalog resources in one worker, up to 100 parallel parameter update operations are possible.
Before increasing the concurrency, check the PostgreSQL connections and the limits of external APIs.

## Disk space sizing

The demo installation uses the following persistent volume sizes of the managed instances:

| Component | Volume size in the demo installation |
| --- | --- |
| PostgreSQL | 8 GB |
| Valkey | 8 GB |
| ClickHouse | 16 GB |
| Total | 32 GB |

The demo installation configuration sets these sizes. They do not grow with the number of users or workers.
When the data grows, increase the volumes or switch to a profile with dedicated databases.
When the databases are hosted in the cluster, additionally account for storage replication and backups.

### PostgreSQL disk

The payload volume `D` is made up of:

- The current catalog: `number of entities × 2 KiB`.
- Events: `number of entities × 5 changes × 5 days × 8 KiB`.
- Process and action runs: `number of users × 2 runs × 129 working days × 256 KiB`.
- The daily history transfer queue: `number of entities × 5 changes × 1 KiB`.

In the model, `D` is doubled for indexes and the space that record updates consume.
For the write-ahead log (WAL), the sizing adds the larger of two margins:
8 GB or twice the daily volume of written data.
The daily write volume is made up of catalog changes
`number of entities × 5 × (2 + 8 + 1) KiB`
and run data `number of users × 2 × 256 KiB` per working day.
The disk budget is `(2 × D + WAL margin) / 0.7`, rounded up to a power of two.
Dividing by 0.7 leaves 30% of free space.

For 1000 users, the payload takes about 88.83 GB.
With indexes, updates, WAL and free space, about 271 GB are required.
After rounding, the budget is 512 GB.

This sizing covers the data of the given scenario.
AI dialogs, large process contexts, additional relations and more detailed logs increase the volume.

The event retention `events.ttl`, `96h` by default, does not define the retention of all DDP data.
The 180 days for process runs are a planning horizon, not an automatic deletion period.

If the transfer of the history to ClickHouse is delayed by more than a day,
increase the space margin for the queue in PostgreSQL proportionally to the delay.
For WAL, also consider the actual settings and possible replica lag.

### Redis disk

The payload volume `B` includes the event streams, the task queue and the temporary results.
How to size their volume is described in the [Redis memory section](#redis-cpu-and-memory).
The disk budget provides three copies of the volume `1.3 × B`:
the persisted data and the space for the snapshot, the AOF log and its rewrite.
The sizing additionally adds one hour of writes at the peak rate of events and task completions.

With `p` events and `q` task completions per second, the hourly write volume is
`(p × 8 KiB + q × (16 + 64) KiB) × 3600`.
The disk budget is `(3 × 1.3 × B + hourly write volume) / 0.7`,
rounded up to a power of two, with a minimum of 8 GB.
For the 1000-user profile, the sizing gives about 79 GB and is rounded up to 128 GB.

The budget is sized for enabled persistence with regular AOF log rewrites.
With a different RDB/AOF policy, rare rewrites or long writes at the peak rate,
recalculate the space for the AOF log and temporary files.
The Redis disk does not replace memory: streams, queues and results must fit in RAM.

### ClickHouse disk

The history volume is `E × Δ × T × S`, where:

- `E` is the number of entities.
- `Δ` is the number of changes of individual properties of one entity per day.
- `T` is the retention period in days.
- `S` is the average size of a change record.

A repeated synchronization of a value that has not changed does not count as a property change.
By default, the history is kept for 180 days.
The `clickhouse.propertyHistory.retentionDays` parameter sets the period.

The profiles assume 5 property changes per entity per day and a record size of 1 KiB before compression.
The compression gain is not taken into account in the sizing.
The disk budget includes 16 GB of base margin, as in the demo installation,
the space for the history and its temporary copy during maintenance, and 30% of free space.
The resulting budget is `(16 + 2 × history volume) / 0.7` GB,
rounded up to a power of two:

| Number of entities | Number of history records over 180 days | Volume of records before compression | ClickHouse disk budget |
| --- | --- | --- | --- |
| 1000 | 900,000 | About 0.92 GB | 32 GB |
| 10,000 | 9,000,000 | About 9.2 GB | 64 GB |
| 20,000 | 18,000,000 | About 18.4 GB | 128 GB |
| 50,000 | 45,000,000 | About 46.1 GB | 256 GB |
| 100,000 | 90,000,000 | About 92.2 GB | 512 GB |

Select a volume no smaller than the calculated budget.
If the change rate, the record size or the retention period changes, recalculate the disk budget.
Compression reduces the actual data volume.
Part merges and cleanup lag increase the temporary disk consumption.

### DDP temporary disk

Allocate temporary disk space on the nodes separately.
The API and the workers use `emptyDir` temporary volumes, including for the cache and for working with repositories.
The module request for `ephemeral-storage` is about 0.052 GB per container and covers only logs.
Cloned repositories, template packs and container images are not included in this request.
