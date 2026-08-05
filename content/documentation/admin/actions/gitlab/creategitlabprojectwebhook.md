---
title: CreateGitlabProjectWebhook
weight: 30
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateGitlabProjectWebhook — creates a webhook in a GitLab project.

### Request example

```yaml
project_id: '0'
url: https://example.com
push_events: true
issues_events: true
merge_requests_events: true
pipeline_events: true
```

### Request specification

| Name                    | Required           | Description                                                    |
| ----------------------- | ------------------ | ----------------------------------------------------------------- |
| project_id              | Yes                 | Identifier of the project in which to create the webhook          |
| url                     | Yes                 | Webhook URL                                                        |
| push_events             | Yes                 | Trigger the webhook when changes are pushed to the repository      |
| issues_events           | Yes                 | Trigger the webhook when an issue is created                       |
| merge_requests_events   | Yes                 | Trigger the webhook when a merge request is created                |
| pipeline_events         | Yes                 | Trigger the webhook when a pipeline is run                          |

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/hooks`.
