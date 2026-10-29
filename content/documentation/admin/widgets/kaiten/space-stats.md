---
title: Kaiten. Space statistics
description: Configuration and metrics provided by the Kaiten space statistics widget.
weight: 20
---

The widget provides aggregated metrics and statistics for cards in a Kaiten space over a selected period.
Use it to analyze team performance and identify bottlenecks in business processes.

## Configuration

When adding the widget, specify the following parameters:

| Name     | Required | Description                        | Default value |
|----------|----------|------------------------------------|---------|
| Space ID | Yes      | Kaiten space identifier            | —       |

## Query parameters

The widget supports the following query parameters:

| Name           | Required | Description                       | Default value |
|----------------|----------|-----------------------------------|-------------|
| Created after  | Yes      | Start date of the analysis period | 1 month ago |
| Created before | Yes      | End date of the analysis period   | Now         |

## Displayed data

The widget contains four tabs.

### General metrics

Primary metrics:

- **Queued**: Tasks waiting to be processed.
- **Completed**: Completed tasks.
- **In progress**: Active tasks.

Additional metrics:

- **Blocked**: Number of blocked tasks.
- **Blocking**: Number of tasks that block other tasks.
- **Archived**: Number of archived tasks.
- **Urgent**: Number of urgent tasks.
- **Average completion time**: Average time to complete a task, in minutes.

Checklist statistics:

- **Total with checklists**: Total number of tasks that have checklists.
- **Checklist completed**: Tasks with fully completed checklists.
- **Checklist incomplete**: Tasks with incomplete checklists.

### By user

The statistics by user include the following data:

- List of users and their assigned task counts.
- Progress bar visualization.
- Number of tasks assigned to each user.

### Forgotten tasks

Cards that have not been updated since they were created.

### Recently updated

The ten most recently updated cards.

## Authorization

Authorization is described in [External services](../../external-services/#kaiten).
