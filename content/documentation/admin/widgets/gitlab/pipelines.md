---
title: GitLab. Pipelines
description: Configuration and manual start settings for GitLab pipelines.
weight: 50
---

The widget displays data about GitLab pipelines.

## Configuration

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Additional widget features

### Starting pipelines

The widget lets you start GitLab pipelines directly from Deckhouse Development Platform (DDP).

#### Configuration

| Name      | Required | Description                                                   | Default value |
| --------- | -------- | ------------------------------------------------------------- | ------------- |
| Ref       | Yes      | Target branch or tag on which to start the pipeline           | —             |
| Variables | No       | Key-value variables to pass to the pipeline being started     | —             |

### Managing pipelines and jobs

A pipeline row can be expanded to show its stages and jobs. The following actions are available for a pipeline and for an individual job:

- **Retry pipeline** and **Cancel pipeline**;
- **Run job**, **Retry job** and **Cancel job**.

The set of available actions depends on the current state: only a running object can be cancelled, and only a finished one can be retried.

### Job logs

For a job the widget shows the execution log: the last lines inline in the job row and the full log in a separate panel. ANSI escape sequences are rendered with their colors preserved. If the job has never run, the widget says so instead of showing an empty log.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.
