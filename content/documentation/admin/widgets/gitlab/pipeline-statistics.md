---
title: GitLab. Pipeline statistics
description: Pipeline metrics, breakdowns, request parameters, and limits for GitLab projects.
weight: 40
---

The widget displays GitLab pipeline statistics,
including overall statistics and breakdowns by status, source, contributor, and branch.

## Configuration

When adding the widget, specify the following parameters:

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Request parameters

The widget supports the following request parameters:

| Name       | Required | Description                                                                                     | Default value |
| ---------- | -------- | ----------------------------------------------------------------------------------------------- | ------------- |
| Start date | Yes      | Start date of the pipeline analysis period in ISO 8601 format. Example: `2024-01-01T00:00:00Z` | —             |
| End date   | Yes      | End date of the pipeline analysis period in ISO 8601 format. Example: `2024-01-31T23:59:59Z`   | —             |
| Branch     | No       | Filters by a specific branch. If omitted, the widget analyzes all branches                     | —             |

## Displayed data

The widget displays the following statistics:

### Key metrics

The widget displays the following key metrics:

- **Total pipelines**: Total number of pipelines during the selected period.
- **Success rate**: Percentage of successful pipelines.
- **Failure rate**: Percentage of failed pipelines.
- **Average duration**: Average pipeline execution time.

### Breakdown by status

The breakdown by status shows the following data:

- Successful pipelines.
- Failed pipelines.
- Canceled pipelines.
- Skipped pipelines.
- Manual pipelines.

### Breakdown by source

The breakdown by source shows the following data:

- Push events (commits).
- Merge requests.
- Scheduled runs.
- Web interface.

### Top contributors

The top contributors section shows the following data:

- Contributors who started the most pipelines.
- Contributor avatars, if available.
- Number of pipelines for each contributor.

### Branch activity

The widget shows the branches with the most pipelines and the number of pipelines for each branch.

## Limitations

The widget has the following limitations:

- To optimize performance, the widget analyzes no more than 100 pipelines per request.
- Statistics include only pipelines with valid data, including a status and execution time.
- Data is updated each time the widget refreshes.

## Authorization

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.
