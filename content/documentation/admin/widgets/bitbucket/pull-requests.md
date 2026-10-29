---
title: Bitbucket. Pull requests
description: Configuration, status filtering, and actions for Bitbucket pull requests.
weight: 10
---

The widget displays Bitbucket Pull Request (PR) data and provides actions for managing PRs.

## Configuration

When adding the widget, specify the following parameters:

| Name          | Required | Description                                                    | Example                                                                      |
| ------------- | -------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Project key   | Yes      | The part of the repository URL immediately after `/projects/` | For `https://<BITBUCKET_HOST>/projects/MYTEAM/repos/backend`, specify `MYTEAM` |
| Repository identifier | Yes | The part of the repository URL immediately after `/repos/`    | For `https://<BITBUCKET_HOST>/projects/MYTEAM/repos/backend`, specify `backend` |

where:

- `<BITBUCKET_HOST>`: The hostname of the Bitbucket server.

## Filtering by status

The widget can filter displayed Pull Requests by status.
Select one of the following statuses in the widget request settings:

- **Open**: Displays only open PRs.
- **Merged**: Displays only merged PRs.
- **Declined**: Displays only declined PRs.
- **All**: Displays PRs in any status.

By default, the widget displays only open PRs.

## Additional widget features

The widget provides the following actions:

- **Merge**: Merges an open Pull Request. This action is available only for open PRs.
- **Close**: Declines a Pull Request.
- **Changes**: Displays the diff for a Pull Request.
- **Comments**: Displays and adds comments to a PR.
- **Create PR**: Creates a Pull Request with a source branch, target branch, reviewers, title, and description.

{{< alert level="info" >}}
Pull Request actions require the corresponding permissions in the Bitbucket repository.
{{< /alert >}}

## Authorization

Authentication is described in [External services](../../external-services/#bitbucket).
