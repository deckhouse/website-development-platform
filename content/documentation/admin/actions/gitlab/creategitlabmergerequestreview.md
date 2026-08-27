---
title: CreateGitlabMergeRequestReview
weight: 45
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateGitlabMergeRequestReview — creates a merge request in GitLab for a branch that already exists.

Unlike [CreateGitlabMergeRequest](../creategitlabmergerequest/), this action does not create the branch and checks whether a merge request for the same work is already open. If one is found, no new merge request is created and the action returns a link to the existing one.

### Request example

```yaml
skipReview: false
sourceBranch: ddp/template-upgrade-1.2.0
targetBranch: main
dedupBranchPrefix: ddp/template-upgrade-
repositoryRef:
  gitlabProjectId: "0"
mergeRequestSpec:
  title: Upgrade template
  description: Apply template upgrade changes
```

### Request specification

| Name                            | Required | Description                                                                                     |
| ------------------------------- | -------- | ----------------------------------------------------------------------------------------------- |
| skipReview                      | No       | When `true`, the action succeeds without creating a merge request                                |
| sourceBranch                    | Yes      | Existing branch that holds the changes                                                           |
| targetBranch                    | Yes      | Branch the changes are proposed to                                                               |
| dedupBranchPrefix               | No       | Branch prefix used to detect duplicates: an open merge request on a branch with this prefix blocks creating a new one |
| repositoryRef.gitlabProjectId   | Yes      | GitLab project identifier                                                                        |
| mergeRequestSpec.title          | Yes      | Merge request title                                                                              |
| mergeRequestSpec.description    | No       | Merge request description                                                                        |

### Note

Duplicate detection works as follows:

- if an open merge request exists for exactly `sourceBranch`, the action returns it and succeeds;
- if `dedupBranchPrefix` is set and an open merge request exists for another branch with that prefix, the action fails so that parallel updates of the same kind are not created.

The action is used by the [service template upgrade](../../../templates/#обновление-сервиса) processes.
