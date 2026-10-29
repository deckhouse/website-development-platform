---
title: Releasing a new template version and upgrading microservices
weight: 41
description: Releasing a new DDP microservice template version and carrying the changes over to microservice repositories — local upgrade check, manual upgrade with a button, automatic upgrade, and resolving conflicts.
---

This guide describes releasing version `1.1.0` of the `go-service` template and upgrading the microservices created from version `1.0.0`.
In the new version, CI changes and a `.dockerignore` file is added.
As a result, merge requests with the template changes are opened in the microservice repositories, and after they are merged, the microservices' template version in the catalog becomes `1.1.0`.

This guide continues the [Creating and publishing a microservice template](template-first-pack/) example.

## Before you begin

To follow this guide, prepare the following:

- The `go-service` template version `1.0.0` in the template registry and microservices created from it.
- The "Upgrade microservice from template" process bound to the template in the "Template actions" block of the mapping rules.
- The `edit:template-registries` permission to change the template registry settings.

## Preparing a new version

Make changes to the pack:

1. Change `.gitlab-ci.yml`, for example, add a test job.
1. Create a `.dockerignore` file.
1. Add `.dockerignore` to the `update.addIfMissing` list of the manifest:

   ```yaml
   update:
     strategy: selective_additive
     overwrite:
       - .gitlab-ci.yml
       - deploy/**
     addIfMissing:
       - .editorconfig
       - .dockerignore
   ```

1. Validate the pack with the `ddp-template lint ./go-service` command.

Files that are not included in `update.overwrite` and `update.addIfMissing`, for example, `cmd/{{ .slug }}/main.go`, are not changed on upgrade, so developers' changes to these files are preserved.

## Checking the upgrade on a copy of a microservice

Before publishing, check which files the upgrade will change in an existing microservice.
Copy the `values` section of the microservice's `.ddp/template.lock.yaml` lock file to a `service-values.yaml` file and render the new version, simulating the upgrade strategy:

```shell
ddp-template render ./go-service \
  --values ./service-values.yaml \
  --use-overwrite \
  --output ./render-out
```

The values of parameters with the `x-ddp-from` annotation are already written to the lock file, so the command renders the files with the microservice's data.

## Publishing and synchronizing

Publish version `1.1.0` and load it into the portal:

1. Publish the version:

   ```shell
   ddp-template publish ./go-service --version 1.1.0 \
     --registry registry.example.com \
     --repository-prefix ddp-templates
   ```

1. Go to "Administration" → "Integrations" → "Templates" → "Sync runs", click "Sync", and wait for the "Success" status.
1. On the "Templates" tab, make sure the "Latest" tag marks version `1.1.0`.

## Upgrading manually

Upgrade a single microservice with the button on its card:

1. Open the Home page → "Microservices" and select a microservice created from `go-service`.
1. On the "Overview" tab, in the "Version 1.1.0 available" block, click "Upgrade".
1. In the "Run process" dialog, click "Run" and wait for the process to complete in the "My activities" panel.
1. Open the microservice project in GitLab and find the `Upgrade go-service to 1.1.0` merge request from the `ddp/template-upgrade-1.1.0` branch.

The merge request contains the changed `.gitlab-ci.yml`, the new `.dockerignore`, and the lock file with version `1.1.0`.
If `.dockerignore` already existed in the repository, it is not changed.

Merge the merge request.
After the version reconciliation interval, the "Version" field on the microservice card changes to `1.1.0`, and the "Version available" block disappears.
Version reconciliation runs if an automation user is set in the registry settings; without it, the version in the catalog is updated the next time the upgrade process runs.

## Upgrading automatically

Automatic upgrade opens merge requests in all of the template's microservices without clicking a button.
Configure it before publishing the next version.

Configure the automation user:

1. Create a service user, for example, `template-bot`, and fill in the GitLab token and username in its profile.
1. Go to "Administration" → "Integrations" → "Templates" → "Settings".
1. Select the user in the "Automation user" field.
1. Set "Upgrade queue rate limit" to at least the number of the template's microservices, for example, `20`, and save the settings.

Enable automatic upgrade for the microservices.
To do this, open the microservice entity in the catalog and set the "Microservice template update policy" property to "Auto review".

After the next version, for example, `1.2.0`, is published and synchronized, the portal runs the upgrade process for each microservice with the "Auto review" policy on behalf of the automation user.
Merge requests are created the same way as with a manual upgrade.

Automatic upgrade does not run for a microservice if its repository has an open merge request with an upgrade to the previous version.
Merge or close that merge request and upgrade the microservice with the "Upgrade" button.

## Conflicts and manual changes

The `selective_additive` strategy overwrites `.gitlab-ci.yml` and the `deploy/**` files entirely.
If developers changed these files in the repository, their changes are replaced with the template version in the merge request; review the changes before merging.

To preserve developers' changes in template files, set the `merge` strategy for the microservice:

1. Open the microservice entity in the catalog.
1. Set the "Microservice template upgrade strategy" property to `merge`.
1. Run the upgrade with the "Upgrade" button.

With the `merge` strategy, the portal merges the template changes with the changes in the repository.
If the changes overlap, the upgrade fails with the `merge_conflict` error listing the files, and the repository is not changed.
In this case, switch back to the `selective_additive` strategy, run the upgrade, and move the developers' changes to the merge request branch manually.

## Deleting files from a template

The upgrade does not delete files.
If a file is deleted or renamed in a new template version, the old file remains in the microservice repositories.

List the files that are no longer used in the template version description, and delete them in the upgrade merge request branch before merging it.
