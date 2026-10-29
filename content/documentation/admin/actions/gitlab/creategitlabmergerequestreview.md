---
title: CreateGitlabMergeRequestReview
weight: 75
---

{{< alert level="info" >}}
Running this action requires the `password` credentials: the GitLab token of the user under which the action runs.
{{< /alert >}}

CreateGitlabMergeRequestReview — creates a merge request in GitLab from an existing branch.
In the interface, the action is called "Create Gitlab merge request for an existing branch".

Unlike [CreateGitlabMergeRequest](creategitlabmergerequest/), the action does not create a branch or write files; instead, it checks the project's open merge requests before creating one.
The action is used by the [microservice template upgrade](../../templates/updates/) process.

## Request example

Example of the action request body:

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

## Request specification

The action request body supports the following parameters:

| Name                            | Required | Description                                                                                                                                                             | Default value                                                                                 |
| ------------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `skipReview`                    | No       | If set to `true`, the action completes successfully without creating a merge request                                                                                    | `false`                                                                                       |
| `sourceBranch`                  | Yes      | Existing branch with the changes                                                                                                                                        | —                                                                                             |
| `targetBranch`                  | No       | Branch to which the changes are proposed. If not set and the action runs for a microservice entity, the value of the `microservice_default_branch` mapping role is used | `main`                                                                                        |
| `dedupBranchPrefix`             | No       | Branch prefix used to search for open merge requests                                                                                                                    | The "Upgrade branch prefix" value of the template registry, otherwise `ddp/template-upgrade-` |
| `repositoryRef.gitlabProjectId` | Yes      | GitLab project ID. If not set and the action runs for a microservice entity, the microservice project ID is used                                                        | —                                                                                             |
| `mergeRequestSpec.title`        | No       | Merge request title                                                                                                                                                     | `Upgrade template (<sourceBranch>)`                                                           |
| `mergeRequestSpec.description`  | No       | Merge request description                                                                                                                                               | —                                                                                             |

## How it works

Deckhouse Development Portal (DDP):

1. Requests the project's open merge requests whose branches start with the `dedupBranchPrefix` prefix.
1. If a merge request from the `sourceBranch` branch is open, completes the action successfully and returns this merge request.
1. If a merge request from another branch with the same prefix is open, fails the action with the `review_already_open: open review for branch <branch> blocks <sourceBranch>` error.
1. Otherwise, creates a merge request from `sourceBranch` to `targetBranch`.

One page of 100 open merge requests of the project is checked.

## Response

The action response contains the following fields:

| Name           | Description                                                                             |
| -------------- | --------------------------------------------------------------------------------------- |
| `skipped`      | `true` if the action completed without creating a merge request because of `skipReview` |
| `deduplicated` | `true` if an already open merge request was returned                                    |
| `provider`     | Version control system provider, `gitlab`                                               |
| `reviewId`     | Merge request number (`iid`) in the project                                             |
| `reviewUrl`    | Link to the merge request                                                               |
| `sourceBranch` | Source branch                                                                           |
| `targetBranch` | Target branch                                                                           |

## GitLab API request

The action performs GET and POST requests to the URL: `/api/v4/projects/:id/merge_requests`.
A merge request creation error is returned with the `review_create_failed` code.
