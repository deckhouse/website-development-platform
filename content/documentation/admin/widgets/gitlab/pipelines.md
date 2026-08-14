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

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.
