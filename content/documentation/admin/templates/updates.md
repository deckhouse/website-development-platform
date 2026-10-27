---
title: Template upgrades
weight: 40
description: How DDP upgrades microservice repositories to a new template version — template binding, manual and automatic upgrades, upgrade strategies, the merge request, version reconciliation, and troubleshooting.
---

A template upgrade transfers the changes of a new template version to the repository of a microservice created from this template.
The portal renders the new pack version with the microservice parameter values, applies the changes according to the upgrade strategy in a separate branch, and opens a merge request.
The changes reach the main branch after the microservice owner accepts the merge request.

## Before you begin

To upgrade from a template, make sure the following conditions are met:

- The [template registry](registry/) is connected and synced.
- In the Home page mapping rules, a process for the "Upgrade microservice from template" action is bound to the template: "Microservices" → "[Template actions](../home/mapping/#template-actions)".
  The "Microservice template registry" dataset creates the "Upgrade microservice from template" process, which can be bound to any template.
- The microservice was created from a template: the repository contains the `.ddp/template.lock.yaml` lock file, and the template binding properties are filled in for the microservice entity.

## Binding a microservice to a template

When a microservice is created from a template, the portal writes the template information to the repository and to the catalog.

The `.ddp/template.lock.yaml` lock file is written to the repository:

```yaml
templateId: go-service
templateVersion: 1.2.0
schemaVersion: 1
values:
  port: 8080
  database: postgres
```

The lock file contains the parameter values passed at creation, without the default values from the schema.
The portal uses the lock file to determine the current template version in the repository and the parameter values for rendering a new version.
Do not delete or edit the lock file manually, because the repository cannot be upgraded without it.

The binding properties are written to the microservice entity.
The table lists the properties of the canonical catalog model and the mapping roles through which the portal finds them:

| Property | Name | Role | Value |
| --- | --- | --- | --- |
| `template_id` | "Template ID" | `microservice_binding_template_id` | Template ID |
| `template_version` | "Template version" | `microservice_binding_template_version` | Current template version in the repository |
| `template_source` | "Template source" | `microservice_binding_template_source` | Name of the registry the template was obtained from |
| `template_update_policy` | "Microservice template update policy" | `microservice_binding_template_update_policy` | "Manual" (`manual`) or "Auto review" (`review-auto`) |
| `template_upgrade_strategy` | "Microservice template upgrade strategy" | `microservice_binding_template_upgrade_strategy` | Upgrade strategy for this microservice |
| `template_upgrade_overwrite` | "Microservice upgrade overwrite paths" | `microservice_binding_template_upgrade_overwrite` | List of overwritten paths for this microservice |
| `template_upgrade_add_if_missing` | "Microservice upgrade add-if-missing paths" | `microservice_binding_template_upgrade_add_if_missing` | List of added paths for this microservice |
| `default_branch` | — | `microservice_default_branch` | Branch that the merge request with the upgrade targets |
| `vcs_provider` | — | `microservice_vcs_provider` | Version control system provider |
| `vcs_external_service_uuid` | "VCS external service" | `microservice_vcs_external_service_uuid` | GitLab external service that hosts the repository |

A microservice is bound to a "template and registry" pair: upgrades are offered only from versions of the template in the same registry.

### Upgrade policy

The upgrade policy defines whether an upgrade starts automatically after a new template version is released.

With the "Manual" (`manual`) policy, an upgrade starts only with the "Upgrade" button in the microservice card.
This policy is assigned when a microservice is created.

With the "Auto review" (`review-auto`) policy, the portal starts an upgrade on its own after the sync that brought a new latest version, and opens a merge request.

To change the policy, open the microservice entity in the catalog and change the "Microservice template update policy" property.

### Microservice upgrade settings

The upgrade strategy and path lists are defined in the template pack manifest.
For an individual microservice, you can override them with the "Microservice template upgrade strategy", "Microservice upgrade overwrite paths", and "Microservice upgrade add-if-missing paths" entity properties.

The portal selects upgrade settings in the following order:

1. Upgrade process parameters, if they are filled in at run time.
1. Microservice entity properties, if they are filled in.
1. The `update.strategy`, `update.overwrite`, and `update.addIfMissing` fields of the new template version manifest.
1. The `selective` strategy, if the strategy is not set anywhere.

The "Upgrade" button and automatic upgrades pass the selected settings in the process launch context.
If the "Upgrade microservice from template" process from the dataset is run without a context, for example, from the catalog, and the "Upgrade strategy" parameter is empty, `selective` is used.

## Detecting a new version

A new template version is detected during a template registry sync.
If the sync changes the latest template version, the portal starts an automatic upgrade of the matching microservices.

In the microservice card on the Home page, the "Version {version} is available" notification is shown when the following conditions are met:

- The template is set for the microservice entity.
- The template in the same registry has a latest version with the "available" status.
- The latest version is higher than the microservice template version.
- The user has the `read:template-packs` permission.

Notifications about prerelease versions, for example, `2.0.0-rc.1`, are visible only to users with the `read:template-registries` permission.
Downgrading to an earlier version is not offered.

## Manual upgrade

A user starts a manual upgrade from the microservice card on the Home page.
A manual upgrade is available with any upgrade policy.

To upgrade a microservice, follow these steps:

1. Open the "Microservices" section of the Home page and select the microservice in the table.
1. On the "Overview" tab, in the "Version {version} is available" block, click "Upgrade".
1. In the "Run process" window, check the parameters and click "Run".
1. Wait for the process to finish and open the merge request in GitLab.
   The link to the merge request is saved in the process store.

The "Upgrade" button is unavailable in the following cases:

| Tooltip | Cause |
| --- | --- |
| "You do not have permission to run processes on this entity." | The user does not have permission to run actions for the microservice entity |
| "A template upgrade process is already running for this service." | An upgrade process is already running for the microservice |
| "No upgrade process is set for the {template} template. Set it in the Home page mapping rules, in the template actions." | No process for the "Upgrade microservice from template" action is bound to the template |
| "The service has no template, so upgrading from a template is unavailable." | No template is set for the microservice entity |

## Automatic upgrade

An automatic upgrade runs the upgrade process without user involvement after a sync that brought a new latest version of the template.

The portal starts an automatic upgrade of a microservice if all of the following conditions are met:

- The "Automation user" is set in the template registry settings.
- The microservice has the "Auto review" (`review-auto`) upgrade policy.
- The new latest version is stable, not a prerelease.
- The microservice's template and registry match the template and registry of the new version.
- The microservice template version is in semver format and is lower than the new version.
- The microservice repository is hosted in GitLab.
- A process for the "Upgrade microservice from template" action is bound to the template.
- The upgrade process is bound to the microservice resource in the "Resource" field of the process.
- No upgrade process is running for the microservice.
- The repository has no open merge request from a branch with the prefix from the "Upgrade branch prefix" setting.

After a single sync, no more upgrades start than the number set in the "Upgrade queue rate limit" setting.
Upgrades with the largest version gap start first: first by major version, then by minor and patch.

{{< alert level="warning" >}}
Microservices that did not fit within the limit or were skipped because of an open merge request are not queued for an upgrade again until the next template version is released.
Upgrade such microservices with the "Upgrade" button or increase the "Upgrade queue rate limit" before the next version is published.
{{< /alert >}}

An automatic upgrade also starts after a sync triggered with the "Sync" button, including when the "Enable sync" parameter is off.
Deprecating, restoring, and deleting a version on the "Templates" tab do not start an automatic upgrade.

### Automation user

Automatic upgrade processes run on behalf of the automation user, and the process actions are performed with this user's credentials.

To prepare the automation user, follow these steps:

1. Create a user, for example, a service user.
   An automatic process run does not check the user's permissions on microservice entities.
1. In the user's profile, fill in the GitLab credentials — a personal access token and a username — for the GitLab external service that hosts the microservice repositories.
1. Select the user in the "Automation user" field on the "Settings" tab of the template registry.

The automation user's credentials are also used to check for open merge requests and to reconcile the template version.

During automatic upgrades, merge requests are created on behalf of the automation user.
Grant this user's GitLab account only the permissions required to create branches and merge requests in microservice repositories.

## Upgrade execution

The "Upgrade microservice from template" process from the "Microservice template registry" dataset consists of two tasks: "Upgrade repository from a template pack" and "Create code review".

The "Upgrade repository from a template pack" task performs the following steps:

1. Determines the target version: the version from the "Template version" parameter or the latest available template version from the microservice registry.
1. Clones the main branch of the repository and reads the lock file.
   If the version in the lock file differs from the version in the catalog, the portal corrects the version in the catalog according to the lock file.
1. If the version in the lock file is not lower than the target version, completes the task without changes. No merge request is created.
1. Renders the new pack version.
   Parameter values: the default values from the new version schema, overridden by the values from the lock file, overridden by the values passed to the process.
1. Creates the `<UPDATE_BRANCH_PREFIX><VERSION>` branch, for example, `ddp/template-upgrade-1.3.0`, from the main branch.
1. Applies the changes according to the upgrade strategy.
1. Writes the new version and parameter values to the lock file.
1. Creates the `chore(template): upgrade <id> <old version> -> <new version>` commit and pushes the branch to GitLab.

The "Create code review" task opens a merge request from the upgrade branch with the `Upgrade <TEMPLATE_ID> to <VERSION>` title.
The merge request targets the branch from the microservice's `default_branch` property or, if the property is not set, `main`.

If a merge request from the same branch is already open, the task uses it.
If a merge request from another branch with the same prefix is open, the task fails with the `review_already_open` error.

The portal does not accept the merge request automatically, does not assign reviewers, and does not send notifications.
The process result is shown in the "My activities" panel and in the process run history.

### Upgrade strategies

The upgrade strategy defines which repository files an upgrade changes. Strategies and path list requirements:

| Strategy | `overwrite` | `addIfMissing` | Behavior |
| --- | --- | --- | --- |
| `selective` | Required | Not used | Overwrites files whose paths match `overwrite` |
| `additive` | Not used | Required | Adds files from `addIfMissing` if they are missing from the repository |
| `selective_additive` | Required | Required | First overwrites files from `overwrite`, then adds files from `addIfMissing` |
| `full` | Optional | Optional | Writes the entire rendered pack over the repository and overwrites every matching file |
| `merge` | Required | Optional | Performs a three-way merge of files from `overwrite`, then adds files from `addIfMissing` |

Rules for all strategies:

- Files outside the strategy scope are not changed.
- Files are not deleted: files deleted or renamed in the new template version remain in the repository.
- `template.yaml`, `values.yaml`, and `.templateignore` are not copied to the repository.

Path lists are glob patterns relative to the repository root. The `**` syntax is supported, for example, `deploy/**`.

If a list required by the strategy is empty, the upgrade fails with the `invalid_upgrade_config` error.

### Merging

The `merge` strategy preserves the changes that developers made to files from `overwrite`.
For each file, the portal compares three versions:

- Base: The file rendered from the template version specified in the lock file.
- Current: The file in the microservice repository.
- New: The file rendered from the new template version.

If the file is missing from the repository, the new version of the file is written.
CRLF line endings and the trailing newline at the end of the file are preserved.

Any merge conflict, including a difference in binary files, aborts the entire upgrade.
Nothing is written to the repository, and the process fails with the `merge_conflict: merge conflict in files: <files>` error.
To upgrade a microservice with conflicts, set another strategy for it, for example, `selective`, and transfer the changes to the merge request manually.

## Version reconciliation after merging

After the merge request is accepted, the portal updates the microservice template version in the catalog.
Reconciliation runs on the schedule from the "Template version reconcile interval" setting.

Reconciliation runs if the "Enable sync" parameter is on in the registry settings and the "Automation user" is set.
The portal checks microservices whose template version in the catalog is lower than the latest available version, reads the lock file from the `default_branch` branch (or `main`) through the GitLab API, and raises the "Template version" property if the version in the lock file is higher.

The version in the catalog can only be raised.
After reconciliation, the "Version {version} is available" notification disappears from the microservice card.

If the automation user is not set, the version in the catalog does not change after the merge request is accepted until the next upgrade run, because the upgrade process corrects the version according to the lock file.

## Troubleshooting

Upgrade errors are shown in the process task log. The message starts with an error code.

| Error code | Cause | Action |
| --- | --- | --- |
| `lock_not_found` | The repository has no `.ddp/template.lock.yaml` file | Restore the lock file. Repositories created without a lock file cannot be upgraded from a template |
| `lock_mismatch` | The lock file was created for a different template | Check the "Template ID" property of the microservice and the lock file |
| `lock_invalid`, `unsupported_lock_schema` | The lock file is corrupted or has an unsupported schema version | Restore the lock file from the repository history |
| `merge_conflict` | Merge conflict with the `merge` strategy | Change the microservice strategy or perform the upgrade manually |
| `invalid_upgrade_config` | A path list required by the strategy is not set | Set the path lists in the pack manifest or in the microservice properties |
| `review_already_open` | A merge request from another upgrade branch is open | Accept or close the open merge request and retry the upgrade |
| `push_failed` | Failed to push the branch to GitLab, for example, a branch with the same name already exists | Delete the upgrade branch in GitLab and retry the upgrade |
| `pack_not_found` | The microservice registry has no suitable template version | Check the microservice template registry and the version status |
| `pack_deprecated`, `pack_unavailable`, `pack_invalid` | The target version is deprecated, unavailable, or invalid | Select another version or restore the version on the "Templates" tab |
| `values_invalid` | The parameter values do not match the new version schema | Pass the missing values to the upgrade process or fix the schema in the new template version |
| `render_failed` | Pack file templating error | Fix the templates and publish a new version |
| `clone_failed` | Failed to clone the repository | Check the GitLab credentials of the user who ran the process |
| `unsupported_vcs_provider` | The microservice version control system provider is not GitLab | — |

To retry the upgrade, run the process again with the "Upgrade" button.
