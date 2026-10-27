---
title: CreateGitlabTag
weight: 50
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateGitlabTag — creates a new tag in a GitLab project.

## Request example

Example of the action request body:

```yaml
project_id: '0'
tag_name: v1.0.0
ref: main
message: Tag description
```

## Request specification

The action request body supports the following parameters:

| Name                   | Required | Description                                                                      |
| ---------------------- | -------- | ------------------------------------------------------------------------------------ |
| `project_id`           | Yes      | Identifier of the project in which to create the tag                                  |
| `tag_name`             | Yes      | Name of the tag                                                                        |
| `ref`                  | Yes      | Name of the branch or tag, or the commit SHA, that the new tag will point to           |
| `message`              | No       | Tag description                                                                        |

## GitLab API request

The action performs a POST request to the URL: `/api/v4/projects/:id/repository/tags`.
