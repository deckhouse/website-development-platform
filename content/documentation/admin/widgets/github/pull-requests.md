---
title: GitHub. Pull Requests
weight: 20
---

The widget displays Pull Requests (PRs) for a GitHub repository. It also provides actions for viewing changes and creating, merging, and closing PRs.

## Authentication

Authentication is described in [External services](../../external-services/#github).
In the external service settings or widget configuration, set **URL** to `https://api.github.com`.

## Account and action initiator

Requests to GitHub use the token from the credentials of the platform user on whose behalf the action is invoked. If **Select an account for the widget** is enabled in the widget settings, the selected platform user's credentials are used instead of the current user's credentials.

When a Pull Request is created, merged, or closed, GitHub identifies the GitHub account that owns the token as the PR author and action initiator. The login and name displayed in GitHub may differ from the name in the Deckhouse Development Platform (DDP) profile.

## Configuration

| Name             | Required | Description                                           | Example                                                       |
| ---------------- | -------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| Repository owner | Yes      | Repository owner, either an organization or a user.   | For `https://github.com/example/my-repo`, specify `example`.   |
| Repository       | Yes      | Repository name without the `.git` suffix.            | For `https://github.com/example/my-repo`, specify `my-repo`.   |

## Status

Filter PRs by status in the widget request settings:

- **Open** — displays only open PRs that are not drafts.
- **Draft** — displays only drafts.
- **Closed** — displays only closed PRs.
- **All** — displays PRs in any status.

By default, the widget displays open PRs. The table displays the number, title, description, status, labels, author, creation date, and update date. An action menu is available for each PR.

## Actions

- **Changes** — displays the list of changed files and the diff for each file.
- **Merge** — merges an open PR. This action is available only for open PRs that are not drafts.
- **Close** — closes a PR without merging it.
- **Create PR** — creates a Pull Request. In the dialog, specify the title, source branch, target branch, and description.

{{< alert level="info" >}}
Pull Request actions require the corresponding permissions in the GitHub repository.
{{< /alert >}}
