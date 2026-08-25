---
title: CreateRepositoryFromTemplatePack
weight: 55
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateRepositoryFromTemplatePack — fills a repository with the contents of a [template pack](../../../templates/) from the registry.

The action pulls the requested template version, renders values into its files and commits the result to the target repository. It also writes `.ddp/template.lock.yaml` with the template identifier and version — the platform later uses that file to decide whether the service can be upgraded.

### Request example

```yaml
targetRepositoryUrl: https://gitlab.example.com/group/service.git
targetBranch: main
templateId: go-service
templateVersion: 1.2.0
commitMessage: Create repository from template
values:
  serviceName: billing-api
```

### Request specification

| Name                     | Required | Description                                                              |
| ------------------------ | -------- | -------------------------------------------------------------------------- |
| targetRepositoryUrl      | Yes      | HTTPS address of the repository the result is written to                  |
| targetBranch             | No       | Branch to commit to; defaults to the branch from the template registry settings |
| templateId               | Yes      | Template identifier in the registry                                       |
| templateVersion          | Yes      | Template version                                                          |
| templatePackSource       | No       | Registry name, when the same template is published in several registries  |
| values                   | No       | Template values; the field set is described by the pack itself            |
| additionalIgnorePatterns | No       | Extra paths excluded from templating                                      |
| commitMessage            | No       | Commit message                                                            |

### Note

The `targetRepositoryUrl` host is validated against the external service configured on the action, so credentials cannot be sent to an unrelated repository.
