---
title: CreateGitlabGroupVariables
weight: 90
---

{{< alert level="info" >}}
Running this action requires the token of the user on whose behalf it will be run.
{{< /alert >}}

CreateGitlabProjectVariables — creates project-level variables in GitLab.

### Request example

```yaml
project_id: '0'
variables:
  - key: EXAMPLE_VARIABLE
    value: value
```

### Request specification

| Name              | Required | Description                                                                     |
| ----------------- | -------- | ---------------------------------------------------------------------------------- |
| project_id        | Yes      | Identifier of the project in which to create the variables                          |
| variables         | Yes      | List of variables to create                                                         |

The field list for variables corresponds to the official GitLab Project-level CI/CD variables API, `/projects/:id/variables`, for more information see [the GitLab documentation](https://docs.gitlab.com/api/project_level_variables/#create-a-variable).

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/variables`.
