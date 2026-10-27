---
title: Quick Home page setup
weight: 30
description: Step-by-step setup of the DDP Home page on a new installation — the initial setup wizard, enabling the Home page, credentials, the template registry, binding processes to templates, and verifying the result.
---

This guide describes setting up the Home page on a new Deckhouse Development Portal (DDP) installation with the canonical catalog model.
As a result, the Home page shows the cluster's environments and deployments, and developers create microservices from templates and deploy them to environments.

## Before you begin

To follow this guide, prepare the following:

- A portal user with the `apply:seeds`, `update:home-mapping-sets`, `edit:template-registries`, and `read:template-packs` permissions, for example, with the "Admin" role preset.
- The Kubernetes API server URL and a service account token with permissions to create namespaces and deployments.
- The Vault or Deckhouse Stronghold URL and an access token.
- The GitLab URL, a GitLab personal access token with permissions to create projects, and a GitLab username.
- A container registry with published template packs.
  Publishing a pack is described in the [Creating and publishing a microservice template](template-first-pack/) example.

## Applying datasets

Datasets create the catalog model, mapping rules, connections to external systems, datasources, and Home page processes.

To apply the datasets:

1. Go to "Administration" → "Directories" → "Datasets" and click "Initial setup".
1. At the "Preset" step, select "Platform setup".
   To populate the catalog with examples, select "Platform setup with demo data".
1. At the "Select" step, keep all datasets selected and click "Next".
1. At the "Parameters" step, fill in the required parameters:
   - "Kubernetes API URL" in the "Kubernetes" dataset, for example, `https://api.cluster.example.com`
   - "Vault URL" in the "Vault" dataset, for example, `https://vault.example.com`
   - "GitLab URL" in the "GitLab" dataset, for example, `https://gitlab.example.com`

   The wizard fills in the remaining parameters automatically.
1. At the "Review" step, enable "Confirm Application" and click "Apply".
1. Wait for the "All selected datasets were applied successfully" message.

If applying a dataset fails, fix the cause and click "Retry from current".

## Enabling the Home page

The Home page is disabled after the portal is installed. To enable it, follow these steps:

1. Go to "Administration" → "Directories" → "Mapping rules".
1. Make sure the table contains the "Home (default)" set with the "Default for Home" switch enabled.
1. Enable the "Show Home" switch.

The "Home" tab appears in the portal's top bar.

## Configuring credentials

Home page processes perform actions in Kubernetes, Vault, and GitLab using the credentials of the user who runs them.

Fill in the credentials in the user profile: the Kubernetes token, the Vault token, and the GitLab token and username.
The procedure is described in the "Credentials" section of the user guide.

Every user who runs processes from the Home page fills in the credentials in their own profile.

## Environments from the cluster

Kubernetes datasources load into the catalog only namespaces, Deployments, and Ingresses with the `ddp.deckhouse.io/environment` label.
Namespaces created with the "Create environment" button on the Home page get the label automatically.

To show an existing namespace on the Home page as an environment, add the label to it with a value equal to the namespace name:

```shell
d8 k label namespace <NAMESPACE> ddp.deckhouse.io/environment=<NAMESPACE>
```

Where `<NAMESPACE>` is the namespace name.

The `d8-*` and `kube-*` namespaces are not loaded.

The environment appears in the "Environments" section after the namespace datasource from the "Kubernetes: deployments, ingresses, and namespaces" dataset is synchronized.
In the catalog, fill in the environment properties that do not come from the cluster: the type, the domain, and the relation to the cluster.

## Connecting the template registry

The template registry connects a container registry that stores template packs. To connect the registry, follow these steps:

1. Go to "Administration" → "Integrations" → "Templates".
1. On the "Registries" tab, click "Connect registry" and fill in the fields:
   - "Name": For example, `company`.
   - "Registry": The registry host, for example, `registry.example.com`.
   - "Repository prefix": For example, `ddp-templates`.
   - "Username" and "Password": The registry credentials.
1. Save the connection.
1. On the "Settings" tab, enable "Enable sync" and save the settings.
1. On the "Sync runs" tab, click "Sync" and wait for the "Success" status.
1. On the "Templates" tab, check that the template versions have the "available" status.

If the sync fails, the error text is shown in the "Error" column.
The causes of errors are described in the [Template registry](../templates/registry/#sync-errors) section.

## Binding processes to templates

Actions on a microservice are performed by processes bound to the microservice template.
The "Microservice template registry" dataset has created processes that work with any template.

Bind the processes to each template:

1. Go to "Administration" → "Directories" → "Mapping rules" and open the "Home (default)" set.
1. On the "Catalog mapping" tab, expand the "Microservices" area and find the "Template actions" block.
1. Click "Add template" and select the template in the "Template" field.
1. Select the action processes:
   - "Create microservice": "Create microservice from template"
   - "Deploy microservice to an environment": "Deploy microservice to an environment"
   - "Delete microservice from an environment": "Delete microservice from an environment"
   - "Upgrade microservice from template": "Upgrade microservice from template"
1. Repeat steps 3–4 for the remaining templates and click "Save".

## Verifying the result

Check that the Home page works by creating a microservice:

1. Open the "Home" tab → "Create microservice".
1. Select a template, fill in the "Service" and "Parameters" steps, and click "Create microservice".
1. In the "Run process" dialog, click "Run".
1. Wait for the process to complete in the "My activities" panel.

The Home page is set up if the following conditions are met:

- A microservice project with the `.ddp/template.lock.yaml` file is created in GitLab.
- The microservice is shown in the "Microservices" section, and the "Template" and "Version" fields are filled in on its card.
- The cluster's environments are shown in the "Environments" section.

## Next steps

After the basic setup, do the following if needed:

- Configure automatic microservice upgrades: [template version release example](template-release-update/).
- Add custom processes to the Home page: the [Custom resource ordering process](home-custom-process/) example.
- Grant developers access to catalog entities: the [Access permissions](../home/overview/#access-permissions) section.
