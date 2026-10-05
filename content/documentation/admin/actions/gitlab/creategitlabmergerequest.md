---
title: CreateGitlabMergeRequest
weight: 70
---

{{< alert level="info" >}}
Running this action requires credentials:

- `password` — the password (token) for the user on whose behalf the action will run.
- `username` — the username of the user on whose behalf the action will run.
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
| source_project_id         | Yes      | Identifier of the template project whose files are added to the Merge Request                                                                 | -                       |
| target_project_id         | Yes      | Identifier of the target project in which the Merge Request will be created                                                                   | -                       |
| merge_request_spec        | Yes      | Specification matching the [GitLab Merge Requests API](https://docs.gitlab.com/ee/api/merge_requests.html#create-mr)                          | -                       |
| source_project_tag        | No       | Tag of the template project to clone. If not specified, the branch `source_project_branch` is used                                           | -                       |
| source_project_branch     | No       | Branch of the template project to clone. It is used only to clone the template                                                               | main                    |
| additionalIgnoreFiles     | No       | List of files containing paths to exclude from the MR. Populated similarly to [.templateignore](createrepositoryfromtemplate/#templateignore) | -                       |
| values                    | No       | Variables used during templating, in `key: value` format                                                                                      | -                       |

### How it works

The portal:

1. Clones the template project (`source_project_id`) at the tag `source_project_tag` or, if the tag is not specified, at the branch `source_project_branch`. For more information, see [Implementation details](createrepositoryfromtemplate/#implementation-details).
1. Reads the `values.yaml` file stored at the root of the template repository and determines the default templating variables.
1. Reads the variables passed when the action is launched and merges them with the variables from `values.yaml`. Variables passed at launch take priority.
1. Reads the `.templateignore` file and determines the directories and files excluded from templating.
1. Renders the files from the templates, taking into account `values.yaml` and the variables passed to the action.
1. Clones the target project (`target_project_id`) at the branch `merge_request_spec.target_branch`.
1. Creates the branch `merge_request_spec.source_branch` from `merge_request_spec.target_branch` in the clone and switches to it.
1. Copies the rendered files to the target project, overwriting files with the same paths.
1. Commits the changes and pushes the commit to the branch `merge_request_spec.source_branch` of the target project. The push is not forced: if this branch already exists in the target project and contains commits that are not in `merge_request_spec.target_branch`, the action fails.
1. Creates an MR in the target project according to `merge_request_spec` by sending a POST request to the GitLab API.

### Note

The action performs a POST request to the URL: `/api/v4/projects/:id/merge_requests`.
