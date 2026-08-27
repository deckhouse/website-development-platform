---
title: UpgradeRepositoryFromTemplatePack
weight: 56
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

UpgradeRepositoryFromTemplatePack — brings the changes of a newer [template pack](../../../templates/) version into a repository.

The action never writes to the main branch: it creates a separate branch with the changes so that they can be reviewed in a merge request. Which files are touched is decided by the upgrade strategy.

### Request example

```yaml
targetRepositoryUrl: https://gitlab.example.com/group/service.git
targetBranch: main
sourceBranch: ddp/template-upgrade-1.3.0
templateId: go-service
templateVersion: 1.3.0
upgradeStrategy: selective
updateOverwrite:
  - .gitlab-ci.yml
updateAddIfMissing:
  - docs/CONTRIBUTING.md
commitMessage: Upgrade template to 1.3.0
```

### Request specification

| Name                     | Required | Description                                                        |
| ------------------------ | -------- | -------------------------------------------------------------------- |
| targetRepositoryUrl      | Yes      | HTTPS address of the service repository                             |
| targetBranch             | No       | Branch the upgrade branch is created from                           |
| sourceBranch             | Yes      | Branch the upgrade changes are written to                           |
| templateId               | Yes      | Template identifier                                                 |
| templateVersion          | Yes      | Template version to upgrade to                                      |
| templatePackSource       | No       | Registry name, when the same template is published in several registries |
| upgradeStrategy          | No       | Upgrade strategy; defaults to the one declared by the pack          |
| updateOverwrite          | No       | Paths that are overwritten                                          |
| updateAddIfMissing       | No       | Paths that are added only when missing                              |
| values                   | No       | Template values                                                     |
| additionalIgnorePatterns | No       | Extra paths excluded from templating                                |
| commitMessage            | No       | Commit message                                                      |

### Note

The upgrade strategies and their behaviour are described in [Upgrading a service](../../../templates/#обновление-сервиса).

The upgrade is atomic: if merging conflicts in at least one file, the action fails with the list of conflicting files and the repository is left untouched. An upgrade never deletes files that are absent from the new template version.
