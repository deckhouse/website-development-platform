---
title: UpgradeRepositoryFromTemplatePack
weight: 14
---

{{< alert level="info" >}}
Running this action requires credentials: `username` (the username under which the action runs)
and `password` (the password or token for that user).
{{< /alert >}}

UpgradeRepositoryFromTemplatePack — transfers the changes of a new [template pack](../../templates/packs/) version to a microservice repository.
In the interface, the action is called "Upgrade repository from a template pack".

The action does not change the main branch.
It creates an upgrade branch with the changes, which are then proposed in a merge request created by the [CreateGitlabMergeRequestReview](creategitlabmergerequestreview/) action.
The upgrade mechanism is described in ["Template upgrades"](../../templates/updates/).

The action runs for a microservice entity.

## Request example

Example of the action request body:

```yaml
templateVersion: 1.3.0
upgradeStrategy: selective
updateOverwrite:
  - .gitlab-ci.yml
updateAddIfMissing:
  - docs/CONTRIBUTING.md
```

## Request specification

Unset `templateId`, `templatePackSource`, `targetRepositoryUrl`, and `targetBranch` fields are filled in from the microservice entity properties by the roles of the [mapping rules](../../home/mapping/#microservices).

| Name                       | Required | Description                                                                         | Default value                                                                                           |
| -------------------------- | -------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `targetRepositoryUrl`      | Yes      | HTTPS address of the microservice repository                                        | The `microservice_repository` role                                                                      |
| `targetBranch`             | No       | Branch from which the upgrade branch is created                                     | The `microservice_default_branch` role, then the "Default branch" of the template registry, then `main` |
| `sourceBranch`             | No       | Branch to which the changes are written                                             | `<UPDATE_BRANCH_PREFIX><VERSION>`, for example, `ddp/template-upgrade-1.3.0`                            |
| `templateId`               | Yes      | Template ID                                                                         | The `microservice_binding_template_id` role                                                             |
| `templatePackSource`       | Yes      | Name of the connection in the template registry                                     | The `microservice_binding_template_source` role                                                         |
| `templateVersion`          | No       | Target template version in semver format                                            | Latest template version                                                                                 |
| `upgradeStrategy`          | No       | Upgrade strategy: `selective`, `additive`, `selective_additive`, `full`, or `merge` | The microservice property,  then the pack strategy, then `selective`                                    |
| `updateOverwrite`          | No       | Glob patterns of paths to overwrite                                                 | The microservice property,  then the pack list                                                          |
| `updateAddIfMissing`       | No       | Glob patterns of paths to add if missing                                            | The microservice property,  then the pack list                                                          |
| `values`                   | No       | Template parameter values. They are merged with the values from the lock file       | —                                                                                                       |
| `additionalIgnorePatterns` | No       | Additional glob patterns of paths that are not rendered or copied                   | —                                                                                                       |
| `commitMessage`            | No       | Commit message                                                                      | `chore(template): upgrade <id> <old> -> <new>`                                                          |

The strategies and the path lists they require are described in ["Upgrade strategies"](../../templates/updates/#upgrade-strategies).

## How it works

Deckhouse Development Portal (DDP):

1. Clones the `targetBranch` branch and reads the `.ddp/template.lock.yaml` lock file.
   If the version in the catalog differs from the version in the lock file, corrects the microservice template version property.
1. If the version in the lock file is not lower than the target version, completes the action successfully without changes.
1. Renders the target pack version with the values from the lock file and the `values` field.
1. Creates the `sourceBranch` branch and applies the changes according to the upgrade strategy. Files are not deleted.
1. Writes the new version and parameter values to the lock file, creates a commit, and pushes the `sourceBranch` branch.

If a merge conflict occurs with the `merge` strategy, nothing is written to the repository.

## Response

The action response contains the following fields:

| Name              | Description                                                         |
| ----------------- | ------------------------------------------------------------------- |
| `templateId`      | Template ID                                                         |
| `templateSource`  | Registry connection name                                            |
| `previousVersion` | Template version before the upgrade                                |
| `targetVersion`   | Target version                                                      |
| `sourceBranch`    | Upgrade branch                                                     |
| `targetBranch`    | Branch from which the upgrade branch was created                    |
| `strategy`        | Applied strategy                                                    |
| `commitSha`       | SHA of the upgrade commit                                           |
| `upgradeSkipped`  | `true` if the repository is already at the target version           |
| `versionDrift`    | Details about the catalog version correction based on the lock file |

Error codes are described in ["Troubleshooting"](../../templates/updates/#troubleshooting).
