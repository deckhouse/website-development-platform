---
title: Kaiten. Space statistics
description: Configuration and metrics provided by the Kaiten space statistics widget.
weight: 20
---

The widget provides aggregated metrics and statistics for cards in a Kaiten space over a selected period.
Use it to analyze team performance and identify bottlenecks in business processes.

## Configuration

| Name     | Required | Description                        | Default |
|----------|----------|------------------------------------|---------|
| Space ID | Yes      | Kaiten space identifier            | —       |

## Query parameters

| Name           | Required | Description                       | Default     |
|----------------|----------|-----------------------------------|-------------|
| Created after  | Yes      | Start date of the analysis period | 1 month ago |
| Created before | Yes      | End date of the analysis period   | Now         |

## Displayed data

The widget contains four tabs.

### General metrics

Primary metrics:

- **Queued** — tasks waiting to be processed.
- **Completed** — completed tasks.
- **In progress** — active tasks.

Additional metrics:

- **Blocked** — number of blocked tasks.
- **Blocking** — number of tasks that block other tasks.
- **Archived** — number of archived tasks.
- **Urgent** — number of urgent tasks.
- **Average completion time** — average time to complete a task, in minutes.

Checklist statistics:

- **Total with checklists** — total number of tasks that have checklists.
- **Checklist completed** — tasks with fully completed checklists.
- **Checklist incomplete** — tasks with incomplete checklists.

### By user

- List of users and their assigned task counts.
- Progress bar visualization.
- Number of tasks assigned to each user.

### Forgotten tasks

Cards that have not been updated since they were created.

### Recently updated

The ten most recently updated cards.


## Authorization

Authorization is described in [External services](../../external-services/#kaiten).
