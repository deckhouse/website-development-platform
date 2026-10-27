---
title: GetGitlabPipeline
weight: 105
---

{{< alert level="info" >}}
Running this action requires the token of the user under which it runs.
{{< /alert >}}

GetGitlabPipeline — returns the state of a GitLab pipeline.

The action is used in processes that start a pipeline with the [StartGitlabPipeline](startgitlabpipeline/) action and wait for it to complete by periodically requesting its state.

## Request example

Example of the action request body:

```yaml
project_id: "0"
pipeline_id: "0"
```

## Request specification

The action request body supports the following parameters:

| Name          | Required | Description                                                                                                                      |
| ------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `project_id`  | Yes      | GitLab project ID or project path, for example, `group/project`                                                                  |
| `pipeline_id` | Yes      | Pipeline ID, that is, the `id` field of the GitLab response, for example, `{{ .response.id }}` of the StartGitlabPipeline action |

## Response

The action response is the GitLab pipeline object with the `id`, `iid`, `status`, `ref`, `sha`, `web_url`, and other fields.
The pipeline state is available in subsequent process elements as `{{ .response.status }}`.

## GitLab API request

The action performs a GET request to the URL: `/api/v4/projects/:id/pipelines/:pipeline_id`.
