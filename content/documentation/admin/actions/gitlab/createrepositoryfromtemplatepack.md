---
title: CreateRepositoryFromTemplatePack
weight: 12
---

{{< alert level="info" >}}
Running this action requires credentials: `username` (the username under which the action runs)
and `password` (the password or token for that user).
{{< /alert >}}

CreateRepositoryFromTemplatePack — populates a repository with the files of a [template pack](../../templates/packs/) from the template registry.
In the interface, the action is called "Create repository from a template pack".

The action downloads a pack version, renders the files with the passed parameter values, writes the `.ddp/template.lock.yaml` lock file to the repository, and pushes a commit to the target branch.
The portal then uses the lock file to [upgrade the repository](../../templates/updates/) to new template versions.

## Request example

Example of the action request body:

```yaml
targetRepositoryUrl: https://gitlab.example.com/group/billing-api.git
targetBranch: main
templateId: go-service
templateVersion: 1.2.0
templatePackSource: company
values:
  port: 8080
```

## Request specification

The action request body supports the following parameters:

| Name                       | Required | Description                                                                                                   | Default value                                                         |
| -------------------------- | -------- | ------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `targetRepositoryUrl`      | Yes      | HTTPS address of the repository. The host must match the host of the action's external service                | —                                                                     |
| `targetBranch`             | No       | Branch to which the commit is pushed                                                                          | The "Default branch" value of the template registry, otherwise `main` |
| `templateId`               | Yes      | Template ID                                                                                                   | —                                                                     |
| `templatePackSource`       | Yes      | Name of the connection in the [template registry](../../templates/registry/) from which the template is taken | —                                                                     |
| `templateVersion`          | No       | Template version in semver format                                                                             | Latest template version                                               |
| `values`                   | No       | Template parameter values. The set of parameters is defined by the pack's `values.yaml` file                  | —                                                                     |
| `additionalIgnorePatterns` | No       | Additional glob patterns of paths that are not rendered or copied to the repository                           | —                                                                     |
| `commitMessage`            | No       | Commit message                                                                                                | `chore(template): scaffold from <templateId>@<version>`               |

## How it works

Deckhouse Development Portal (DDP):

1. Selects the template version in the `templatePackSource` registry. The version must have the "available" status.
1. Downloads the pack and fills in the parameters annotated with `x-ddp-from` from the [process run context](../../home/processes/#creating-a-microservice).
1. Validates the parameter values against the `values.yaml` schema and renders the pack files.
1. Clones the target branch of the repository. An empty repository is initialized.
1. Copies the rendered files and writes the `.ddp/template.lock.yaml` lock file with the template ID, template version, and parameter values.
1. Creates a commit and pushes it to the target branch.

## Response

The action response contains the `templateId`, `templateVersion`, `templateSource`, `targetBranch`, and `commitSha` fields, as well as `values.request` (the passed parameter values) and `values.merged` (the resulting values including defaults).

## Errors

An error message starts with a code. The main codes are:

| Code                          | Cause                                                                                                                |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `invalid_request`             | `templateId` or `templatePackSource` is not set, or the version is not in semver format                              |
| `pack_unavailable`            | The `templatePackSource` connection is not found, or the version is unavailable                                      |
| `pack_not_found`              | The template version is not found                                                                                    |
| `pack_deprecated`             | The template version has been withdrawn                                                                              |
| `values_invalid`              | The parameter values do not match the `values.yaml` schema, or the `x-ddp-from` path is not found in the run context |
| `render_failed`               | Pack file templating failed                                                                                          |
| `clone_failed`, `push_failed` | Cloning the repository or pushing the commit failed                                                                  |
