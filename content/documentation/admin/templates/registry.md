---
title: Template registry
weight: 20
description: Connecting container registries with template packs to DDP, sync and automatic upgrade settings, sync runs, the template version catalog, and version statuses.
---

The template registry is a set of portal connections to container registries where template packs are published, together with their sync settings.
The portal syncs the connected registries, maintains a catalog of template versions, and determines the latest version of each template.

The template registry is located in "Administration" → "Integrations" → "Templates".
The page contains the "Registries", "Settings", "Sync runs", and "Templates" tabs.

## Quick start

To connect a registry and get templates in the Home page gallery, follow these steps:

1. On the "Registries" tab, click "Connect registry", fill in the "Name", "Registry", "Repository prefix", "Username", and "Password" fields, and save the connection.
1. On the "Settings" tab, turn on the "Enable sync" toggle and save the settings.
1. On the "Sync runs" tab, click "Sync" and wait for the "Success" status.
1. On the "Templates" tab, make sure that template versions have the "available" status.
1. Bind processes to templates in the Home page mapping rules: "Microservices" → "[Template actions](../home/mapping/#template-actions)".

After that, the templates are displayed in the "Create microservice" section of the Home page.

## Access permissions

The following permissions are used to work with the template registry:

| Permission | Grants |
| --- | --- |
| `read:template-packs` | Viewing template versions: the "Create microservice" gallery and the new version notification in the microservice card |
| `edit:template-packs` | Deprecating, restoring, and deleting template versions |
| `read:template-registries` | Viewing connections, settings, and sync runs; notifications about prerelease template versions in the microservice card |
| `edit:template-registries` | Changing connections and settings, starting a sync |

Without the `read:template-registries` permission, the page shows the "You do not have permission to manage the template registry." message.

## Connecting a registry

Container registry connections are added on the "Registries" tab with the "Connect registry" button.
The portal supports registries that implement OCI Distribution Spec v2, for example, Harbor, GitLab Container Registry, and Docker Registry.

Connection fields:

| Name | Description |
| --- | --- |
| "Name" | Unique registry name, up to 128 characters. Written to microservices created from this registry's templates |
| "Registry" | Container registry host, for example, `registry.vendor.example`. If no scheme is specified, HTTPS is used |
| "Repository prefix" | Path prefix inside the registry, for example, `ddp-templates` |
| "Repository IDs" | IDs of the templates that the portal tracks in the registry |
| "Username" | Registry username |
| "Password" | Registry password or token. If no username is set, the value is used as a Bearer token |
| "Skip TLS verification" | Only for lab environments and self-signed certificates. Allows connecting over HTTP |

The password is stored encrypted and is not returned to the interface.
To keep the current password when editing a connection, leave the mask in the "Password" field unchanged.

The portal uses the [trusted certificates](../trusted-certificates/) added to the portal when connecting to the registry.

The portal does not accept the following in the "Registry" field:

- The `http://` scheme unless the "Skip TLS verification" option is enabled
- Credentials, query parameters, and fragments in the address
- `localhost` and `*.localhost` names
- IP addresses of loopback, private, link-local, and multicast networks

### Tracked templates

The "Repository IDs" field defines how the portal finds templates in the registry.

If the field is empty, the portal requests the registry's repository catalog with the prefix from the "Repository prefix" field.
New templates appear automatically, and a template deleted from the registry is deprecated entirely.

If the field is filled in, the portal requests only the listed repositories.
This mode suits large registries and tokens without permission to view the catalog.
New templates are added to the list manually, and a template deleted from the registry remains in the gallery.

In both modes, versions deleted from tracked repositories are deprecated.

A template ID is the repository name in the registry without the prefix.
A pack is published at `<REGISTRY>/<REPOSITORY_PREFIX>/<TEMPLATE_ID>:<VERSION>`.

### Deleting and renaming a connection

When a connection is deleted or renamed, the available versions of its templates get the "unavailable" status with the `source removed from settings` reason.
Version records are not deleted from the catalog.

If a connection with the same name is added again, these versions become available on the next sync.

Versions of deleted connections are shown on the "Templates" tab with the "Registry removed" tag.
The "Delete orphaned ({count})" button deletes them permanently. To confirm, enter `DELETE`.

## Settings

The "Settings" tab contains parameters shared by all connections. Changes are applied without restarting the portal.

The "Synchronization" section:

| Name | Default value | Description |
| --- | --- | --- |
| "Enable sync" | Disabled | Enables scheduled sync and reconciliation of the microservice template version with the lock file |
| "Sync interval" | `*/15 * * * *` | Cron expression for the sync schedule, in UTC |
| "Max template size (kilobytes)" | `51200` | A pack larger than the limit is rejected during sync |

The "Defaults for created services" section:

| Name | Default value | Description |
| --- | --- | --- |
| "Default branch" | `main` | Default branch for created microservices |
| "Upgrade branch prefix" | `ddp/template-upgrade-` | Prefix of the template upgrade branch. The version is appended to it, for example, `ddp/template-upgrade-1.3.0`. Ends with `-` or `/`, contains no spaces or `..`, up to 128 characters |

Changing the upgrade branch prefix does not take into account merge requests already opened from a branch with the old prefix.

The "Version reconcile and automation" section:

| Name | Default value | Description |
| --- | --- | --- |
| "Template version reconcile interval" | `*/5 * * * *` | Cron expression for the microservice template version reconciliation schedule |
| "Automation user" | Not set | User on whose behalf automatic upgrades run and version reconciliation is performed. If not set, automatic upgrades and version reconciliation are disabled |
| "Upgrade queue rate limit" | `5` | Maximum number of automatic upgrades started after a single sync. A value from 1 to 1000 |

Requirements for the automation user are described in ["Template upgrades"](updates/#automatic-upgrade).

## Sync

Sync loads new template versions from the connected registries into the version catalog.

Sync runs on the schedule from the "Sync interval" parameter if sync is enabled,
or when you click "Sync" on the "Sync runs" tab.
The button starts a sync even when the "Enable sync" parameter is off.

Only one sync runs at a time.
Starting another sync while one is running fails with the "Template registry sync is already running" error.

For each connection, sync performs the following steps:

1. Gets the list of repositories from the "Repository IDs" field or from the registry catalog.
1. For each semver tag that is not yet in the version catalog, downloads the pack and checks its size, the `template.yaml` manifest, the `values.yaml` schema, and the `minDdpVersion` requirement.
1. Writes the version to the catalog with the "available" status or with the reason why the version is unavailable.
1. Deprecates versions whose tags were deleted from the registry.
1. Recalculates the latest version of each template.

Tags that are not in strict semver format, for example, `v1.2.0` or `latest`, are skipped.
Versions that have already been loaded are not loaded again.

If the latest version of a template has changed, an [automatic upgrade](updates/#automatic-upgrade) of microservices starts after the sync.

A connection error for one registry, for example, an authorization error, interrupts the sync of that registry but does not affect the sync of the others.
In this case, the run gets the "Error" status.

A sync that runs longer than 2 hours is marked as failed with the `interrupted: sync run exceeded stale threshold` error when the portal restarts.

### Sync runs

The "Sync runs" tab shows the last 10 runs.
The summary of the last run is displayed above the table, for example, "Last successful synchronization: {time}" or "The last synchronization run failed.".

Table columns:

- "Status": "Running", "Success", or "Error".
- "Time": Run start time.
- "Changes": Number of added, deprecated, conflicting, orphaned, and restored versions, or "No changes".
- "Error": Error text.

The "Refresh" button reloads the list of runs.

## Version catalog

The "Templates" tab shows all template versions loaded from the connected registries.

For each version, the tab shows the template, version, name, registry, status, status reason, sync time, image reference, and SHA-256.
The "Latest" tag marks the recommended template version.

### Version statuses

A template version can have the following statuses:

| Status | Assigned when | Effect on the version |
| --- | --- | --- |
| "available" | The pack is loaded and has passed the checks | Shown in the gallery and used for creation and upgrades |
| "invalid" | The pack could not be downloaded, unpacked, or validated | Not used. The reason is shown in the "Status reason" column |
| "unavailable" | The version requires a newer Deckhouse Development Portal (DDP) version set in `minDdpVersion`, or the connection was deleted | Not used |
| "withdrawn" | The tag was deleted from the registry, or an administrator deprecated the version | Hidden from the gallery and automatic upgrades |

The portal does not recheck versions with the "invalid" status or versions unavailable because of `minDdpVersion`.
To make the portal load such a version again, delete it on the "Templates" tab. On the next sync, the tag is loaded from the registry again.
To fix a pack, publish a new version.

### Latest version

The latest template version is the highest stable version with the "available" status across all connections.
If there are no stable versions, the highest prerelease version, for example, `2.0.0-rc.1`, becomes the latest.

The latest version is not always the highest version according to semver rules, because deprecated versions are not taken into account.

### Registry conflict

If the same template version exists in several connections, the version from the connection that loaded it first is used.
The version shows the "Conflict" tag with a list of the other registries.

If the version from the first connection becomes unavailable or deprecated, an available copy from another connection replaces it.

### Deprecating, restoring, and deleting a version

Managing versions requires the `edit:template-packs` permission.

The following actions are available for a version:

- "Deprecate version": The version gets the "withdrawn" status and is hidden from the gallery and automatic upgrades.
  Specify the reason in the "Reason" field.
  The tag is not deleted from the registry.
- "Restore version": A deprecated version becomes available again.
  If the tag is missing from the registry, the next sync may deprecate the version again.
- "Delete": The version record is deleted from the portal.
  The tag remains in the registry, and the next sync loads the version again.

The "invalid" and "unavailable" statuses cannot be changed manually.
Deprecating, restoring, and deleting a version recalculate the latest template version but do not start an automatic upgrade of microservices.

## Sync errors

Sync errors are shown in the "Error" column of the run, and errors of individual versions are shown in the "Status reason" column of the version catalog.

| Message | Cause | Action |
| --- | --- | --- |
| `registry authorization failed` | The registry rejected the credentials | Check the "Username" and "Password" fields of the connection |
| `registry temporarily unavailable` | The registry is unavailable or returned a 429 or 5xx error | Retry the sync later |
| `list repositories: ...` | Failed to get the repository catalog | Check the token's permission to view the catalog or fill in the "Repository IDs" field |
| `pack size N exceeds maxPackSize M` | The pack is larger than the "Maximum template size" parameter | Reduce the pack size or increase the limit, delete the version, and retry the sync |
| `invalid template.yaml: ...` | The manifest failed validation | Fix the manifest and publish a new version |
| `manifest id "a" does not match repository template id "b"` | The `id` field of the manifest does not match the repository name in the registry | Publish the pack to a repository whose name equals `id` |
| `requires DDP version X or newer (running Y)` | The DDP version is lower than the pack's `minDdpVersion` | Update DDP and delete the version so that it is loaded again |
| `tag removed from registry` | The tag was deleted from the registry | — |
| `source removed from settings` | The connection was deleted or renamed | Restore the connection with the same name or delete the orphaned versions |

Template registry API messages:

| Message | Cause |
| --- | --- |
| "Invalid template registry" | The connection fields failed validation, for example, the address points to a private network |
| "Invalid template registry settings" | Invalid cron expression, upgrade branch prefix, or limit |
| "Template registry sync is already running" | Another sync was started before the current one finished |
| "Invalid template version status change" | Attempt to change a version status other than "available" or "withdrawn" |
