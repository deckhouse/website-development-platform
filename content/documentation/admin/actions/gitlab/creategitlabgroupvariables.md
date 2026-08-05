---
title: CreateGitlabGroupVariables
weight: 80
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
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

The fields for each variable correspond to the official GitLab group-level variables API, `/groups/:id/variables`. For details, see the [GitLab documentation](https://docs.gitlab.com/api/group_level_variables/#create-variable).

### Note

The action performs a POST request to the URL: `/api/v4/groups/:id/variables`.
