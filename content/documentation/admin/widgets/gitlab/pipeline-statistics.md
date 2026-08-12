---
title: GitLab. Pipeline statistics
weight: 40
---

The widget displays GitLab pipeline statistics,
including overall statistics and breakdowns by status, source, member, and branch.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.

## Configuration

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | -             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | -             |

## Displayed data

The widget displays the following statistics:

### Key metrics

- **Total pipelines** — total number of pipelines during the selected period.
- **Success rate** — percentage of successful pipelines.
- **Failure rate** — percentage of failed pipelines.
- **Average duration** — average pipeline execution time.

### Breakdown by status

- Successful pipelines.
- Failed pipelines.
- Canceled pipelines.
- Skipped pipelines.
- Manual pipelines.

### Breakdown by source

- Push events (commits).
- Merge requests.
- Scheduled runs.
- Web interface.

### Top members

- Members who started the most pipelines.
- Member avatars, if available.
- Number of pipelines for each member.

### Branch activity

- Branches with the most pipelines.
- Number of pipelines for each branch.

## Request parameters

| Name       | Required | Description                                                                                     | Default value |
| ---------- | -------- | ----------------------------------------------------------------------------------------------- | ------------- |
| Start date | Yes      | Start date of the pipeline analysis period in ISO 8601 format. Example: `2024-01-01T00:00:00Z` | -             |
| End date   | Yes      | End date of the pipeline analysis period in ISO 8601 format. Example: `2024-01-31T23:59:59Z`   | -             |
| Branch     | No       | Filters by a specific branch. If omitted, the widget analyzes all branches                     | -             |

## Limitations

- To optimize performance, the widget analyzes no more than 100 pipelines per request.
- Statistics include only pipelines with valid data, including a status and execution time.
- Data is updated each time the widget refreshes.
