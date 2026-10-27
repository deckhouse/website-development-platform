---
title: Templates from GitLab
description: Workspace template repository structure, GitLab source setup, version publishing, and archiving.
weight: 50
---

A template source pulls OpenTofu projects from GitLab manually or on a schedule.
Changed files become a new published version of the template, which is activated immediately.
Files of such a template are edited in GitLab.

## Preparing the repository

Place each template in a separate immediate subdirectory of the selected directory.
For example, for the `workspace-templates` source path:

```text
workspace-templates/
  go-development/
    versions.tf
    main.tf
    variables.tf
    outputs.tf
    files/
      AGENTS.md
    scripts/
      start.sh
  project-review/
    versions.tf
    main.tf
    variables.tf
    outputs.tf
    files/
      AGENTS.md
```

The template identifiers in the example are `go-development` and `project-review`.
An immediate subdirectory must contain a `.tf` file in its root; nested `files/` and `scripts/` belong to the same template.
Each project must return the [required `ddp_runtime` output](templates/#connecting-to-the-portal).

To migrate a manual template, place its user files in GitLab with the same relative paths.
Do not add `ddp.tf`, `ddp.auto.tfvars.json`, the OpenTofu state, or `.terraform`, which the portal creates.

In each project, set its own [IDE domain, Ingress, Secrets, and storage](network-storage/).
Identifiers of new templates must not be used by other templates in the portal.

## Creating a source

To connect a repository, prepare a GitLab external service and the credentials of a user with access to the repository.
Then follow these steps:

1. Open "AI" → "Workspaces" → "Template sources" and click "Create source".
1. Specify the name, identifier, owner, and team.
1. On the "Repository" tab, select "GitLab external service", "Credentials user", "Project", "Branch or tag", and "Templates folder".
1. Specify the project as `group/project` or a numeric identifier, and the path as `workspace-templates`.
   For the repository root, use `.`.
1. Select "Kubernetes service" and "Namespace" for new templates of this source.
1. On the "Synchronization" tab, set "Schedule (cron)" and enable "Enabled" if you need automatic synchronization.
1. Save the source and click "Sync now".
1. Open "Sync history" and check the result for each directory.

New templates get the Kubernetes connection, namespace, owner, and team of the source.
For an existing template, these settings are changed separately; changing the source does not overwrite them.
Configure the Git connection for cloning the user project and the skills in the template itself.
The source repository and the repository the user works with serve different purposes.

## Updating a version

To publish a change, edit the files in GitLab, create a commit in the selected branch, and run synchronization or wait for the schedule.

If the directory files have changed, the portal publishes a new version labeled with the first 12 characters of the commit SHA and activates it.
New workspaces are created from this version.
Existing workspaces stay on their previous versions until they are updated manually.

A new version is not created if the files have not changed.
If an earlier template version was activated manually, note the "The active version is not the latest one from the repository" mark and check the active version after synchronization.

A disabled source is synchronized only manually.
Files of a template managed by a source remain available for viewing and comparison in the portal.

## Synchronization results

The history shows the time, status, commit, and result for each directory:

| Result | Description |
| --- | --- |
| "Created" | A new directory was found, and a template was created with a published version |
| "Updated" | The files changed, and a new version was published and activated |
| "No changes" | The files match the already published ones |
| "Archived" | The directory was removed from the repository |
| "Restored" | The directory was returned after being archived by the source |
| "Skipped" | The directory was not processed; check the specified reason |

If an identifier conflicts with a manual template or a template of another source, the existing object is not overwritten.
If an error occurs, check the permissions in GitLab, the project, branch, path, file validity, and `ddp_runtime`, then run synchronization again.

## Deleting a directory or source

Deleting a directory from the selected branch archives the corresponding template.
Created workspaces and published versions are kept; new workspaces cannot be created from an archived template.
Returning the directory restores the template if it was archived by the source.

To delete a source, archive its templates first.
After the source is deleted, they remain archived manual templates with their versions.
Data of existing workspaces is not deleted.
