---
title: GitLab. Contributors
description: Charts of GitLab project commit activity by author for a selected branch or tag and period.
weight: 10
---

The widget displays commit activity charts for a GitLab project branch or tag over the selected period.

## Configuration

| Name       | Required | Description                                                              | Default value |
| ---------- | -------- | ------------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                         | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Displayed data

The widget displays the following charts with the number of commits per month:

- the total number of commits in the selected branch or tag;
- a separate chart for each of the 12 authors with the most commits, with the author name, email and number of commits for the period.

The widget header shows the selected period and the branch or tag.

## Request parameters

The widget request parameters set the branch or tag and the period:

| Name | Required | Description                                       | Default value              |
| ---- | -------- | ------------------------------------------------- | -------------------------- |
| Ref  | No       | Branch or tag whose commits the widget analyzes   | Project default branch     |
| from | No       | Start date of the period                          | One year before the current date |
| to   | No       | End date of the period                            | Current date               |

## Limitations

- The charts are built from the 1,000 most recent commits of the period. The widget displays the note "Showing the 1,000 most recent commits".
- The period is limited to 730 days. If the selected period is longer, the widget shortens it to 730 days before the end date.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.
