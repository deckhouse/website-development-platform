---
title: CreateGitlabBranches
weight: 40
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateGitlabBranches — creates new branches in the target repository.

### Request example

```yaml
project_id: '0'
branches:
  - branch: new-branch
    ref: main
```

### Request specification

| Name              | Required | Description                                                                    |
| ----------------- | -------- | -------------------------------------------------------------------------------- |
| project_id        | Yes      | Identifier of the project in which to create the branches                        |
| branches          | Yes      | List of branches to create                                                        |
| branches.branch   | Yes      | Name of the new branch                                                            |
| branches.ref      | Yes      | Name of an existing branch or a commit SHA                                        |

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/repository/branches`. Upon successful creation, GitLab returns information about the created branches.
