---
title: Preparing workspaces
description: Kubernetes and Git connections, limits, network, storage, and permissions for DDP workspaces.
weight: 10
---

Workspaces run an IDE and an AI agent in Kubernetes based on a workspace template.
The template defines objects in the target cluster; the portal manages creation, start, stop, update, and deletion.

## Preparing the installation

Before the first run, prepare the infrastructure and access:

1. Create a Kubernetes [external service](../external-services/) and fill in the credentials for accessing the target cluster.
   Prepare the target namespace.
1. Grant the credentials permissions for the template objects.
   The default template creates a StatefulSet, Service, Ingress, PersistentVolumeClaim, ConfigMap, and Secret.
   Diagnostics also reads Pods, their logs, and events; workspace management changes the number of StatefulSet replicas and connects to containers.
1. Prepare a storage class for the persistent volume.
   The default template uses `ReadWriteOnce`; you can select the class [in the template files](network-storage/#storing-data).
1. Prepare a separate IDE domain, a wildcard DNS record, and TLS.
   The IDE address of each workspace must be reachable from the user's browser.
   For the setup procedure, see [the Ingress and TLS examples](network-storage/#publishing-the-ide).
1. Ensure that target Pods and operation Jobs can reach the portal HTTPS address with a trusted certificate.
   It is used for the OpenTofu state, model requests, MCP, and IDE access checks.
1. If you use private container registries, prepare Secrets for pulling images [in the corresponding namespaces](network-storage/#images-from-a-private-registry).
1. For a template with a repository, create a Git connection and fill in the credentials.
   Users will need an [AI provider](../../user/ai-assistant/#connecting-an-ai-provider).

When DDP is installed with the `development-platform` module, OpenTofu operation Jobs run in the module namespace.
The workspace Pod, volume, and Ingress are created in the cluster and namespace selected in the template.
These locations can differ.

## Installation limits

Before creating the first workspace, save the limits in the "AI" → "Workspaces" → "Workspace templates" section, "Installation limits" block:

| Field | Description |
| --- | --- |
| "Running workspaces per user" | Limits the number of simultaneously running workspaces of a single owner |
| "Running workspaces in total" | Limits the number of simultaneously running workspaces in the installation |

Both values are integers greater than or equal to one.
Changing the limits requires `edit:workspace-policy`, viewing them requires `read:workspace-policy`.
Workspace creation is unavailable until the limits are saved.
Stopped workspaces do not count toward the running limit.

## Selecting credentials

By default, Kubernetes operations use the credentials of the workspace owner, and repository cloning uses the owner's Git token.
The credentials are filled in the user profile for the corresponding external service.

On the "Connection" tab of the template, you can enable "Use a service account" and select an "Account" separately for Kubernetes and Git.
In this case, operations use the credentials of the selected user.

The Git token of the selected account is available to the owner inside the workspace.
Select an account with access to the projects that this template is allowed to work with.

## Granting access

To allow a user to create workspaces, grant them `create:workspaces` and permission to use the template: the global `use:workspace-templates` or the object-level `use:workspace-template`.
Access to the "AI" section requires `view:ai-page`.

Viewing, starting, stopping, and deleting are controlled by [workspace permissions](../security/rbac/#workspaces-and-templates).
Ownership grants permissions when the `owner-is-admin` setting is enabled.

After the preparation, [create and activate a template](templates/), then follow the user procedure for creating a workspace.

## Module parameters

When DDP is installed with the module, you can override the backend address and operation Job resources.
Example of a ModuleConfig `spec.settings` fragment:

```yaml
workspaces:
  backendUrl: https://ddp.example.com
  jobResources:
    cpuMillicores: 500
    memoryBytes: 1073741824
    temporaryBytes: 134217728
```

Without overrides, each Job uses the portal HTTPS address, `500` CPU millicores, `1` GiB of memory, and `128` MiB of temporary storage.
The CPU and memory values set the Job requests and limits.
Container resources of the workspace itself are set in the template.
