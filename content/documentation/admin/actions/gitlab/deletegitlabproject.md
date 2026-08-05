---
title: DeleteGitlabProject
weight: 110
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

DeleteGitlabProject — deletes an existing project in GitLab through the GitLab API. Authentication uses a GitLab token provided in the credentials.

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
