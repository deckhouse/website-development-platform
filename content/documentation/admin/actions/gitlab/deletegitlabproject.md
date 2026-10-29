---
title: DeleteGitlabProject
weight: 110
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

DeleteGitlabProject — deletes an existing project in GitLab through the GitLab API.
Authentication uses a GitLab token provided in the credentials.

## Request example

Example of the action request body:

```yaml
project_id: 0
```

## Request specification

The action request body supports the following parameters:

| Name                        | Required | Description                                          |
| --------------------------- | -------- | -------------------------------------------------------- |
| `project_id`                | Yes      | Identifier of the project to delete                       |

## GitLab API request

The action performs a DELETE request to the URL: `/api/v4/projects/:id`.
