---
title: StartGitlabPipeline
weight: 100
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

StartGitlabPipeline — starts a pipeline in GitLab.

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
| ref                         | Yes      | Name of the branch or tag, or the commit SHA, on which the pipeline will run         |
| variables                   | No       | List of variables to pass to the pipeline being run                                 |
| variables.key               | Yes      | Variable name                                                                        |
| variables.value             | Yes      | Variable value                                                                       |

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/pipeline`.
