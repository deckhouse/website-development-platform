---
title: StartGitlabPipeline
weight: 100
---

{{< alert level="info" >}}
Running this action requires the token of the user on whose behalf it will be run.
{{< /alert >}}

StartGitlabPipeline — starts a pipeline run in GitLab.

### Request example

```yaml
project_id: 0
ref: main
variables:
  - key: example-key
    value: example-value
```

### Request specification

| Name                        | Required | Description                                                                    |
| --------------------------- | -------- | ---------------------------------------------------------------------------------- |
| project_id                  | Yes      | Identifier of the project in which to run the pipeline                              |
| ref                         | Yes      | Name of the branch, a tag, or a commit SHA hash on which the pipeline will run      |
| variables                   | No       | List of variables to pass to the pipeline being run                                 |
| variables.key               | Yes      | Variable name                                                                        |
| variables.value             | Yes      | Variable value                                                                       |

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/pipeline`.
