---
title: GitHub. Pull requests
description: Status filtering and repository actions for GitHub pull requests.
weight: 20
---

The widget displays Pull Requests (PRs) for a GitHub repository.
It also provides actions for viewing changes and creating, merging, and closing PRs.

## Account and action initiator

Requests to GitHub use the token from the credentials of the Deckhouse Development Portal (DDP) user on whose behalf the action is invoked.
If **Select account for widget** is enabled in the widget settings, the selected portal user's credentials are used instead of the current user's credentials.

When a Pull Request is created, merged, or closed, GitHub identifies the GitHub account that owns the token as the PR author and action initiator.
The login and name displayed in GitHub may differ from the name in the DDP profile.

## Configuration

When adding the widget, specify the following parameters:

| Name             | Required | Description                                           | Example                                                       |
| ---------------- | -------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| Repository owner | Yes      | Repository owner, either an organization or a user   | For `https://github.com/example/my-repo`, specify `example` |
| Repository       | Yes      | Repository name without the `.git` suffix            | For `https://github.com/example/my-repo`, specify `my-repo` |

## Status

Filter PRs by status in the widget request settings:

- **Open**: Displays only open PRs that are not drafts.
- **Draft**: Displays only drafts.
- **Closed**: Displays only closed PRs.
- **All**: Displays PRs in any status.

By default, the widget displays open PRs.
The table displays the number, title, description, status, labels, author, creation date, and update date.
An action menu is available for each PR.

## Actions

The following actions are available for pull requests:

- **Changes**: Displays the list of changed files and the diff for each file.
- **Merge**: Merges an open PR. This action is available only for open PRs that are not drafts.
- **Close**: Closes a PR without merging it.
- **Create PR**: Creates a Pull Request.
  In the dialog, specify the title, source branch, target branch, and description.

{{< alert level="info" >}}
Pull Request actions require the corresponding permissions in the GitHub repository.
{{< /alert >}}

## Authorization

Authentication is described in [External services](../../external-services/#github).
In the external service settings or widget configuration, set **URL** to `https://api.github.com`.
