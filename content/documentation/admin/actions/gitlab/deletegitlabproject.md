---
title: StartGitlabPipeline
weight: 110
---

{{< alert level="info" >}}
Running this action requires the token of the user on whose behalf it will be run.
{{< /alert >}}

DeleteGitlabProject — deletes an existing project in GitLab. The action calls the GitLab API to delete the project. A GitLab token, which must be provided in the credentials, is used for authentication.

### Request example

```yaml
project_id: 0
```

### Request specification

| Name                        | Required | Description                                          |
| --------------------------- | -------- | -------------------------------------------------------- |
| project_id                  | Yes      | Identifier of the project to delete                       |

### Note

The action performs a DELETE request to the URL: `/api/v4/projects/:id`.
