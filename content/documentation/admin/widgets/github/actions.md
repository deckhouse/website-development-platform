---
title: GitHub. Actions
description: Filters and actions for GitHub Actions workflow runs, jobs, logs, and artifacts.
weight: 10
---

The widget displays GitHub Actions runs in a repository. It also displays jobs and artifacts and provides actions for managing them.

## Account and action initiator

Requests to GitHub use the token from the credentials of the Deckhouse Development Portal (DDP) user on whose behalf the action is invoked. If **Select account for widget** is enabled in the widget settings, the selected portal user's credentials are used instead of the current user's credentials.

When a workflow is started, canceled, or restarted, or when artifacts and logs are accessed, GitHub identifies the GitHub account that owns the token as the initiator. The login displayed in GitHub may differ from the name in the DDP profile.

## Configuration

| Name             | Required | Description                                           | Example                                                       |
| ---------------- | -------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| Repository owner | Yes      | Repository owner, either an organization or a user   | For `https://github.com/example/my-repo`, specify `example` |
| Repository       | Yes      | Repository name without the `.git` suffix            | For `https://github.com/example/my-repo`, specify `my-repo` |

## Request parameters

Configure the following filters in the widget request settings:

- **Branch** — displays only runs from the specified head branch.
- **Event** — displays only runs triggered by the selected event type.
- **Status** — displays only runs with the selected status or conclusion.
- **Workflow** — displays only runs for the selected workflow file.
- **Triggered by** — displays only runs started by the specified GitHub user.
- **Created at filter** — displays runs created within the specified start and end dates.

## Actions

The widget provides the following actions:

- **Run workflow** — manually starts a workflow with the `workflow_dispatch` trigger. Select the workflow and branch or tag. If the input YAML declares parameters, the input parameters are displayed.
- **Re-run workflow**, **Re-run failed jobs**, and **Cancel workflow** — manage the selected run.
- **Re-run job** — restarts a completed job with the `failure` or `cancelled` conclusion.

Click a run row to expand its jobs and artifacts. In the expanded row, the **Execution log** button opens the job log, and the **Download** button downloads an artifact. The **Show pipeline** button opens the **Jobs and steps** tree of the run.

{{< alert level="info" >}}
Workflow and artifact actions require the corresponding permissions in the GitHub repository.
{{< /alert >}}

## Authentication

Authentication is described in [External services](../../external-services/#github).
In the external service settings, set **URL** to `https://api.github.com`.
