---
title: CreateGitlabRelease
weight: 60
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateGitlabRelease — creates a GitLab release based on an existing tag.

### Request example

```yaml
project_id: '0'
tag_name: v1.0.0
name: Release v1.0.0
description: |
  ## Changes:
  - New feature.
  - Bug fixes.
```

### Request specification

| Name           | Required | Description                                                                                   |
| -------------- | -------- | ------------------------------------------------------------------------------------------------ |
| project_id     | Yes      | Identifier of the project in which to create the release                                          |
| tag_name       | Yes      | Name of the existing tag on which the release will be based                                       |
| name           | Yes      | Name of the release as displayed in GitLab                                                        |
| description    | No       | Release description in Markdown format                                                           |

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/releases`.
