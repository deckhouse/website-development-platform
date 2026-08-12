---
title: GitLab. Merge requests
weight: 20
---

The widget displays GitLab merge requests (MRs) and provides actions for managing them.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.

## Configuration

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | -             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | -             |

## Status filtering

The widget can filter merge requests by status.
In the widget request settings, select one of the following statuses:

- **Open** — displays only open MRs.
- **Closed** — displays only closed MRs.
- **Merged** — displays only merged MRs.
- **Blocked** — displays only blocked MRs.

By default, the widget displays only open MRs.

## Additional widget features

When actions are enabled in the settings, the widget provides the following merge request actions:

- **Merge** — merges an open merge request. This action is available only for open MRs.
- **Close** — closes a merge request.
- **Mark as draft/ready** — changes the draft status of a merge request.
- **View changes** — displays the diff for a merge request.

{{< alert level="info" >}}
Actions on MRs require the corresponding access permissions in the GitLab repository.
{{< /alert >}}
