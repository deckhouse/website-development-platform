---
title: GitLab. Merge requests
description: Status filtering and management actions for GitLab merge requests.
weight: 20
---

The widget displays GitLab merge requests (MRs) and provides actions for managing them.

## Configuration

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Status filtering

The widget can filter merge requests by status.
In the widget request settings, select one of the following statuses:

- **Open** — displays only open MRs.
- **Closed** — displays only closed MRs.
- **Merged** — displays only merged MRs.
- **Blocked** — displays only blocked MRs.

By default, the widget displays only open MRs.

Above the table the widget shows per-state counters — **All**, **Open**, **Drafts**, **Merged**, **Closed**. A counter also acts as a filter: clicking it switches the table to the selected state.

## Additional widget features

When actions are enabled in the settings, the widget provides the following merge request actions:

- **Create MR** — creates a merge request: specify the source branch, target branch, title and description.
- **Merge** — merges an open merge request. This action is available only for open MRs.
- **Close** — closes a merge request.
- **Mark as draft/ready** — changes the draft status of a merge request.
- **Changes** — displays the diff for a merge request: the list of changed files and a side-by-side comparison of the old and new versions.

When merging is not possible, the widget explains why: the MR is a draft, it conflicts with the target branch, its pipeline is not green, or it is not open.

{{< alert level="info" >}}
Actions on MRs require the corresponding access permissions in the GitLab repository.
{{< /alert >}}

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.

## Placing the widget on the home page

The widget can be shown on the **MRs** tab of a microservice card on the home page. To do that, assign it to the `microservice_merge_requests_widget` role in the home mapping rules.

In that mode the project identifier is not set in the widget configuration by hand: the microservice card passes it through the launch context, and the configuration references it with the `{{ .context.repository.id }}` template.
