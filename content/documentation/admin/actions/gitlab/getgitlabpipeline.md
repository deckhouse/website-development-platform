---
title: GetGitlabPipeline
weight: 105
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

GetGitlabPipeline — returns the state of a GitLab pipeline.

The action is used by processes that start a pipeline with [StartGitlabPipeline](../startgitlabpipeline/) and then wait for it to finish by polling its state.

### Request example

```yaml
project_id: "0"
pipeline_id: "0"
```

### Request specification

| Name        | Required | Description                 |
| ----------- | -------- | --------------------------- |
| project_id  | Yes      | GitLab project identifier   |
| pipeline_id | Yes      | Pipeline identifier         |

### Note

The action sends a GET request to `/api/v4/projects/:id/pipelines/:pipeline_id`. The pipeline response becomes the action result and is available to the following process elements.
