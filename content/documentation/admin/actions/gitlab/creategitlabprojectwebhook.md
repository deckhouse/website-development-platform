---
title: CreateGitlabProjectWebhook
weight: 30
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateGitlabProjectWebhook — creates a webhook in a GitLab project.

## Request example

Example of the action request body:

```yaml
project_id: '0'
url: https://example.com
push_events: true
issues_events: true
merge_requests_events: true
pipeline_events: true
```

## Request specification

The action request body supports the following parameters:

| Name                    | Required           | Description                                                    |
| ----------------------- | ------------------ | ----------------------------------------------------------------- |
| `project_id`            | Yes                 | Identifier of the project in which to create the webhook          |
| `url`                   | Yes                 | Webhook URL                                                        |
| `push_events`           | No                  | Trigger the webhook when changes are pushed to the repository. Defaults to `false` |
| `issues_events`         | No                  | Trigger the webhook when an issue is created. Defaults to `false` |
| `merge_requests_events` | No                  | Trigger the webhook when a merge request is created. Defaults to `false` |
| `pipeline_events`       | No                  | Trigger the webhook when a pipeline is run. Defaults to `false` |

## GitLab API request

The action performs a POST request to the URL: `/api/v4/projects/:id/hooks`.
