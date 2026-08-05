---
title: CreateGitlabTag
weight: 50
---

{{< alert level="info" >}}
Running this action requires the token of the user on whose behalf it will be run.
{{< /alert >}}

CreateGitlabTag — creates a new tag in a GitLab project.

### Request example

```yaml
project_id: '0'
tag_name: v1.0.0
ref: main
message: Tag description
```

### Request specification

| Name                   | Required | Description                                                                      |
| ---------------------- | -------- | ------------------------------------------------------------------------------------ |
| project_id             | Yes      | Identifier of the project in which to create the tag                                  |
| tag_name               | Yes      | Name of the tag                                                                        |
| ref                    | Yes      | Name of the branch, a tag, or a commit SHA hash the new tag will point to             |
| message                | Yes      | Tag description                                                                        |

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/repository/tags`.
