---
title: GitHub. Actions
weight: 10
---

The widget displays GitHub Actions runs in a repository. It also displays jobs and artifacts and provides actions for managing them.

## Authentication

Authentication is described in [External services](../../external-services/#github).
In the external service settings, set **URL** to `https://api.github.com`.

## Account and action initiator

Requests to GitHub use the token from the credentials of the platform user on whose behalf the action is invoked. If **Select an account for the widget** is enabled in the widget settings, the selected platform user's credentials are used instead of the current user's credentials.

When a workflow is started, canceled, or restarted, or when artifacts and logs are accessed, GitHub identifies the GitHub account that owns the token as the initiator. The login displayed in GitHub may differ from the name in the DDP profile.

## Configuration

| Name             | Required | Description                                           | Example                                                       |
| ---------------- | -------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| Repository owner | Yes      | Repository owner, either an organization or a user.   | For `https://github.com/example/my-repo`, specify `example`.   |
| Repository       | Yes      | Repository name without the `.git` suffix.            | For `https://github.com/example/my-repo`, specify `my-repo`.   |

## Request parameters

Configure the following filters in the widget request settings:

- **Branch** — displays only runs from the specified head branch.
- **Event** — displays only runs triggered by the selected event type.
- **Status** — displays only runs with the selected status or conclusion.
- **Workflow** — displays only runs for the selected workflow file.
- **Triggered by** — displays only runs started by the specified GitHub user.
- **Creation date filter** — displays runs created within the specified start and end dates.

## Actions

The widget provides the following actions:

- **Run workflow** — manually starts a workflow with the `workflow_dispatch` trigger. Select the workflow and branch or tag. If the input YAML declares parameters, the input parameters are displayed.
- **Restart workflow**, **Restart failed jobs**, and **Cancel workflow** — manage the selected run.
- **Restart job** — restarts a completed job with the `failure` or `cancelled` conclusion.
- View job logs, download artifacts, open the run on GitHub, and view the job and step tree.

{{< alert level="info" >}}
Workflow and artifact actions require the corresponding permissions in the GitHub repository.
{{< /alert >}}
