---
title: Creating a template
description: Creating and editing a workspace template, creation form parameters, publishing, testing a draft, and activating a version.
weight: 20
---

A workspace template is an OpenTofu project with a Kubernetes connection, Git settings, and a set of skills.
A new manual template contains a starter project with agent, IDE, and access check containers, a persistent volume, a Service, and an Ingress.

## First template

Before creating a template, [prepare workspaces](overview/). Then configure a manual template:

1. Open "AI" → "Workspaces" → "Workspace templates" and click "Create template".
1. Specify a name and a unique identifier, for example, `team-development`.
1. On the "Connection" tab, select "Kubernetes connection" and "Namespace".
   If necessary, select a service account.
1. To work with a project, enable "Initialize with a repository", select "Git connection", and set the HTTPS URL and branch.
   If the user selects the project, enable "Allow overriding on creation".
   For a workspace without Git, leave initialization disabled.
1. Save the template and open the "Files" tab.
1. In `main.tf`, set `local.ide_domain`, for example, `ide.example.com`, and check the [Ingress, TLS, image, and storage parameters](network-storage/).
1. Click "Save", then "Publish v1".
1. On the "Versions" tab, click "Activate v1".

A user creates a new workspace from the active version.
A published version is immutable; the next change is made in a draft and published as a new version.

## Starter project contents

The files define Kubernetes objects and the contents of the working directory:

| Path | Description |
| --- | --- |
| `versions.tf` | OpenTofu version, Kubernetes provider version and configuration |
| `main.tf` | StatefulSet, persistent volume, Secrets, ConfigMap, Service, Ingress, and file delivery settings |
| `variables.tf` | User parameters of the creation form |
| `outputs.tf` | Required `ddp_runtime` output |
| `files/` | Text files for the workspace working directory |
| `scripts/create.sh` | Optional initial setup of a new volume |
| `scripts/start.sh` | Optional setup on every start |

The portal creates `ddp.tf` and `ddp.auto.tfvars.json` itself.
These paths, the `.terraform` directory, and variable names with the `ddp_` prefix are reserved.
Add your own variables without this prefix.

## Creation form parameters

A root module variable becomes a field of the workspace creation form.
In the editor, open `variables.tf` and add a variable:

```hcl
variable "runtime_memory" {
  description = "Agent container memory"
  type        = string
  default     = "2Gi"

  validation {
    condition     = contains(["2Gi", "4Gi", "8Gi"], var.runtime_memory)
    error_message = "Select 2Gi, 4Gi, or 8Gi."
  }
}
```

The default template already contains `runtime_memory`. Replace its block instead of adding a second one.
The value is used in `local.runtime_resources`.
After publishing, the form will offer a choice of memory size; if the user does not change it, they get `2Gi`.

The form supports the `string`, `number`, `bool`, and `list(string)` types.
`description` becomes the field hint, and `default` becomes the initial value.
A variable without `default` is required.
A `contains([...], var.<NAME>)` check defines the options to select from.
Other `validation` conditions are checked by OpenTofu during the operation.

Example of a parameter with a list of Secret names:

```hcl
variable "registry_secrets" {
  description = "Secrets for pulling images in the target namespace"
  type        = list(string)
  default     = []
}
```

For the value to affect the Pod, replace the corresponding line of the existing `locals` block in `main.tf`:

```hcl
image_pull_secrets = var.registry_secrets
```

A parameter with `sensitive = true` is hidden in the interface.
Set Git and Kubernetes connection credentials through the template connections and accounts.

## Testing and publishing changes

To test changes before switching all new workspaces to them:

1. Open the draft and change the files.
1. Save them.
   If another user has changed the draft, refresh it and apply your changes again to the current revision.
1. Click "Check in a workspace".
   The draft is available to users with permission to edit the template; before publishing, default parameter values are used.
1. Check the creation result, IDE, files, and commands in the test workspace.
1. Publish the version and compare it with the previous one using the "Compare" button.
1. Activate the required version on the "Versions" tab.

Activation affects subsequent creations.
Existing workspaces stay on their versions until they are explicitly updated.
To return to the previous template for new creations, activate the previous published version.

## Connecting to the portal

`output "ddp_runtime"` is required in a template. The starter project returns:

```hcl
output "ddp_runtime" {
  value = {
    statefulset    = kubernetes_stateful_set_v1.runtime.metadata[0].name
    namespace      = var.ddp_namespace
    container      = "runtime"
    engine_port    = local.engine_port
    ide_container  = "ide"
    ide_url        = local.ide_url
    work_directory = local.work_directory
  }
}
```

The portal uses the StatefulSet to start and stop the workspace, the agent container and its port for jobs, and the `ide_url` address to open the IDE.
When changing the template, keep the output consistent with the objects actually created and with the IDE HTTPS address.

## Skills and archiving

On the "Skills" tab, select Deckhouse Development Portal (DDP) [skills](../ai-agents/skills/) and a pinned version or "Latest".
The set is fixed when the workspace is created and on every start.

For a template with Git, the "Allow repository skills" toggle controls whether the skills of the working copy itself are available.
If it is disabled, the selected "Hidden skill folders" are excluded from the working copy on start.
This only limits file visibility; the files can still be read through Git.

An archived template keeps its versions and existing workspaces, but you cannot create new workspaces from it.
To resume creation, use "Restore from archive".

## File limits

When editing, take the template limits into account:

| Limit | Value |
| --- | --- |
| Number of files | Up to 256 |
| Size of a single file | Up to 1 MiB |
| Contents | UTF-8 text |
| Path | Relative, without empty segments, `.`, or `..` |

For practical examples, see [file delivery and scripts](files/), [installing Go and other tools](tools/), [images, Ingress, and storage](network-storage/), and [synchronization from GitLab](sources/).
