---
title: CreateGitlabGroupVariables
weight: 80
---

{{< alert level="info" >}}
Running this action requires the token of the user on whose behalf it will be run.
{{< /alert >}}

CreateGitlabGroupVariables — creates group-level variables in GitLab.

### Request example

```yaml
group_id: '0'
variables:
  - key: EXAMPLE_VARIABLE
    value: value
```

### Request specification

| Name              | Required | Description                                                                     |
| ----------------- | -------- | ---------------------------------------------------------------------------------- |
| group_id          | Yes      | Identifier of the group in which to create the variables                            |
| variables         | Yes      | List of variables to create                                                         |

The field list for variables corresponds to the official GitLab Group-level Variables API, `/groups/:id/variables`, for more information see [the GitLab documentation](https://docs.gitlab.com/api/group_level_variables/#create-variable).

### Note

The action performs a POST request to the URL: `/api/v4/groups/:id/variables`.
