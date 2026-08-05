---
title: CreateGitlabMergeRequest
weight: 70
---

{{< alert level="info" >}}
Running this action requires credentials:

- `password` — the password (token) of the user on whose behalf the action will be run.
- `username` — the username on whose behalf the action will be run.
{{< /alert >}}

CreateGitlabMergeRequest — creates a new Merge Request (MR) in the target repository. Files stored in the source repository are added to the Merge Request. Files may contain variables whose values will be substituted at the time the MR is created.

### Request example

```yaml
source_project_id: '0'
source_project_branch: example
source_project_tag: v1.0.0
target_project_id: '0'
merge_request_spec:
  source_branch: example
  target_branch: '1'
  title: example
additionalIgnoreFiles:
  - .ignore
  - .example
values:
  key1: value1
  nested:
    enabled: true
    subkey: 123
```

### Request specification

| Name                      | Required | Description                                                                                                                                   | Default value |
| ------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| source_project_id         | Yes      | Identifier of the project that serves as the source for the Merge Request                                                                     | -                       |
| target_project_id         | Yes      | Identifier of the target project in which the Merge Request will be created                                                                   | -                       |
| merge_request_spec        | Yes      | Specification matching the [GitLab Merge Requests API](https://docs.gitlab.com/ee/api/merge_requests.html#create-mr)                          | -                       |
| source_project_tag        | No       | Tag in the source project from which the Merge Request will be created. If not specified, the source project's branch is used                 | -                       |
| source_project_branch     | No       | Branch in the source project from which the Merge Request will be created                                                                     | main                    |
| additionalIgnoreFiles     | No       | List of files containing paths to exclude from the MR. Populated similarly to [.templateignore](createrepositoryfromtemplate/#templateignore) | -                       |
| values                    | No       | Variables used during templating, in `key: value` format                                                                                      | -                       |

### How it works

The platform:

1. Clones the template repository used to generate the MR, based on its identifier (`source_project_id`). For more information, see [Implementation details](createrepositoryfromtemplate/#implementation-details).
1. Reads the `values.yaml` file stored at the root of the repository and determines the default templating variables.
1. Reads the variables passed when the action is launched and merges them with the variables from `values.yaml`. Variables passed at launch take priority.
1. Reads the `.templateignore` file and determines the directories and files excluded from templating.
1. Renders the files from the templates, taking into account `values.yaml` and the variables passed to the action.
1. Changes the remote of the repository to the target one, according to its ID (`target_project_id`), and performs a git push to the branch in the source project (`source_project_branch`), or to the `main` branch.
1. Creates an MR according to the specified settings by sending a POST request to the GitLab API.

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/merge_requests`.
