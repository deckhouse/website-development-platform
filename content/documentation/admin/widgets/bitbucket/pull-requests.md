---
title: Bitbucket. Pull Requests
description: Configuration, status filtering, and actions for Bitbucket pull requests.
weight: 10
---

The widget displays Bitbucket Pull Request (PR) data and provides actions for managing PRs.

## Configuration

| Name          | Required | Description                                                    | Example                                                                      |
| ------------- | -------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Project key   | Yes      | The part of the repository URL immediately after `/projects/` | For `https://<BITBUCKET_HOST>/projects/MYTEAM/repos/backend`, specify `MYTEAM` |
| Repository ID | Yes      | The part of the repository URL immediately after `/repos/`    | For `https://<BITBUCKET_HOST>/projects/MYTEAM/repos/backend`, specify `backend` |

where:
- `<BITBUCKET_HOST>` — the hostname of the Bitbucket server.

## Filtering by status

The widget can filter displayed Pull Requests by status. Select one of the following statuses in the widget request settings:

- **Open** — displays only open PRs.
- **Merged** — displays only merged PRs.
- **Declined** — displays only declined PRs.
- **All** — displays PRs in any status.

By default, the widget displays only open PRs.

## Additional widget features

When actions are enabled in the settings, the widget provides the following Pull Request actions:

- **Merge** — merges an open Pull Request. This action is available only for open PRs.
- **Close** — declines a Pull Request.
- **View changes** — displays the diff for a Pull Request.
- **Comments** — displays and adds comments to a PR.
- **Create PR** — creates a Pull Request with a source branch, target branch, reviewers, title, and description.

{{< alert level="info" >}}
Pull Request actions require the corresponding permissions in the Bitbucket repository.
{{< /alert >}}

## Authentication

Authentication is described in [External services](../../external-services/#bitbucket).
