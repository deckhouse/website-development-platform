---
title: Images, Ingress, and storage
description: Examples of configuring image pull secrets, a separate IDE domain, Ingress, TLS, cert-manager, and the workspace persistent volume.
weight: 40
---

Settings for images, IDE publishing, and the persistent volume are located in `main.tf` of the starter [workspace template](templates/).
The examples change values in the existing `locals` block; you do not need to add a second block with the same names.

## Images from a private registry

For the workspace Pod to pull private images, create a Secret of the `kubernetes.io/dockerconfigjson` type in the target namespace.
For example, using a prepared Docker config without putting the password on the command line:

```shell
d8 k -n team-workspaces create secret generic workspace-registry --type=kubernetes.io/dockerconfigjson --from-file=.dockerconfigjson=/secure/path/config.json
```

Replace `team-workspaces` with the template "Namespace", and the path with a file that grants access to the required container registry.
The account running the command must have permission to create Secrets in this namespace.

In `main.tf`, replace the value:

```hcl
image_pull_secrets = ["workspace-registry"]
```

For several registries, specify several Secret names.
The starter template adds them to the Pod `imagePullSecrets`, including for pulling the init container image.

Save, publish, and activate the template, then create a workspace.
In "Diagnostics", check that the Pod has pulled the images and the containers are ready.

### Secrets for operation Jobs

OpenTofu Jobs and workspace Pods use separate image pull settings.
When DDP is installed with the module, Jobs run in `d8-development-platform`; additional Secrets for them are set in `spec.settings.imagePullSecrets` of ModuleConfig:

```yaml
imagePullSecrets:
  - operations-registry
```

The `operations-registry` Secret must exist in `d8-development-platform`.
This setting does not create a Secret in the target cluster and does not fill in `local.image_pull_secrets` of the template.
Prepare both locations if private images are used for Jobs and workspaces.

## Publishing the IDE

The IDE of each workspace is published on its own HTTPS host.
For the `ide.example.com` domain, the starter template creates an address like `https://ddp-ws-<SUFFIX>.ide.example.com`.

Before starting, prepare a `*.ide.example.com` wildcard record that points to the ingress controller of the target cluster.
In `main.tf`, set:

```hcl
ide_domain     = "ide.example.com"
ingress_class  = "nginx"
ide_tls_secret = "workspace-ide-tls"
ingress_annotations = {
  "nginx.ingress.kubernetes.io/proxy-read-timeout" = "3600"
  "nginx.ingress.kubernetes.io/proxy-send-timeout" = "3600"
}
```

Replace `nginx` with an existing IngressClass.
The `null` value selects the default class.
The `workspace-ide-tls` Secret must be located in the template namespace and contain a certificate for `*.ide.example.com` with a chain trusted by the browser.

The annotations in the example are intended for ingress-nginx and increase timeouts for long-lived IDE connections.
For another controller, set the annotations it supports.

The created Ingress routes traffic to the `gate` access check container, which admits a session opened through the portal.
Keep this route when changing the Ingress.
Publishing `code-server` directly bypasses the access check.

If the ingress controller already issues a suitable default certificate, set `ide_tls_secret = null`.
Check that the browser trusts the certificate for the workspace host specifically.

## Issuing a certificate with cert-manager

If cert-manager is installed and a ClusterIssuer is configured, you can obtain a separate certificate for each workspace.
Replace the values in `main.tf`:

```hcl
ide_domain     = "ide.example.com"
ingress_class  = "nginx"
ide_tls_secret = "${local.name}-ide-tls"
ingress_annotations = {
  "cert-manager.io/cluster-issuer"                 = "letsencrypt-production"
  "nginx.ingress.kubernetes.io/proxy-read-timeout" = "3600"
  "nginx.ingress.kubernetes.io/proxy-send-timeout" = "3600"
}
```

Replace `letsencrypt-production` with your ClusterIssuer and ensure that the conditions for its domain validation are met.
After creating a workspace, wait for the certificate to be issued and check that the IDE opens through the portal.

Cert-manager can keep the issued Secret after the workspace is deleted.
If certificate Secrets must be deleted automatically together with the Certificate, configure cert-manager with `--enable-certificate-owner-ref`; otherwise, include cleanup of such Secrets in your maintenance procedures.

## Storing data

The starter template creates a persistent volume with the `ReadWriteOnce` mode.
The repository is located in `/workspace/code`, and the agent state is in `/workspace/state`; both paths are on this volume.

To select an existing StorageClass, replace the value in `main.tf`:

```hcl
storage_class = "fast-ssd"
```

The `null` value uses the default StorageClass.
The size is set by the `storage_size` variable in `variables.tf`.
For an initial size of `20Gi` and selectable options, replace the existing block:

```hcl
variable "storage_size" {
  description = "Workspace volume size"
  type        = string
  default     = "20Gi"

  validation {
    condition     = contains(["20Gi", "50Gi", "100Gi"], var.storage_size)
    error_message = "Select 20Gi, 50Gi, or 100Gi."
  }
}
```

A new workspace gets the selected size.
Expanding an existing PVC depends on the StorageClass and cluster capabilities.
Do not reduce the size or change the class of an existing volume: the portal rejects an operation that requires deleting or replacing the volume.

Stopping a workspace keeps the volume; deleting the workspace deletes its persistent data.
Also take into account the reclaim policy of the StorageClass itself.

## Verifying the result

After changing the template, test the draft in a new workspace:

1. On the "Operations" tab, wait for OpenTofu to be applied successfully.
1. In "Diagnostics", check image pulling and the readiness of the containers and the persistent volume.
1. Click "Open IDE" and check the certificate and the terminal.
1. Create a file in the working directory, then stop and start the workspace again. The file must be preserved.

After testing, publish and activate the version.
For existing workspaces, apply a version update.
