---
title: Datasets
menuTitle: Datasets
description: Ready-made configuration sets for integrating the portal with external systems.
weight: 74
---

Datasets are preconfigured configurations for quickly deploying ready-made integrations with external systems and setting up the required components of Deckhouse Development Portal (DDP, portal).

Each dataset contains:

- Dashboards for data visualization
- Widgets for displaying information
- Datasources for synchronization
- External services for connecting to APIs
- Credential types for authentication
- Roles and access permissions
- Other required portal components

## Managing datasets

You can view, apply, and delete datasets in the web interface.

### Viewing available datasets

The "Administration" → "Datasets" section shows a list of all available datasets.
Each card contains:

- "Name": The dataset name.
- "Short description": A general description of the functionality.
- The "Description" button: Opens detailed information about the dataset, including:
  - A detailed description with full information about the components
  - A list of objects that will be created when the dataset is applied
- The "Apply" button: Starts the dataset installation.
- The "Delete" button: Deletes a previously applied dataset.

### Applying a dataset

To apply a dataset, follow these steps:

1. Click "Apply" on the card of the required dataset.
1. If the dataset requires parameter configuration, a configuration form opens.
1. Fill in the required parameters and click "Apply".
1. If no parameters are required, a confirmation window appears.
1. After the dataset is applied successfully, all its components are created in the system.

A dataset that depends on another dataset can be applied only after it. Otherwise, applying fails with the "Failed to apply dataset" error.
If a required parameter is not filled in, the "Fill in all required dataset parameters" error is shown.

### Deleting a dataset

To delete a dataset, follow these steps:

1. Click "Delete" on the card of the applied dataset.
1. Confirm the deletion in the dialog.
1. All dataset components are deleted from the system.

### Configuring after applying

Some datasets may require additional configuration after they are applied:

1. Go to "Profile" → "Credentials".
1. Fill in the required credentials, such as tokens and passwords.
1. Save the changes.

### Initial setup

The "Initial setup" button opens a wizard that applies the "Canonical architecture" datasets for the Home page in a single run: the catalog model, mapping rules, connections to Kubernetes, Vault, and GitLab, datasources, processes, and widgets.
The wizard and the datasets it applies are described in ["Initial setup"](../home/setup/).

The "Show the initial setup banner" toggle returns the "The platform is ready for setup" banner to the Home page after it was hidden with the "Hide" button.

## Available datasets

The following datasets are available in the portal.

### GitLab

This dataset is designed for connecting to the GitLab API.

Created objects:

- The "GitLab" external service
- The "GitLab token" credential type: A personal access token for accessing the GitLab API.
- The "GitLab username" credential type: The GitLab username.

Configuration parameters:

| Name         | Description                                            | Default value         |
|--------------|--------------------------------------------------------|-----------------------|
| "GitLab URL" | Base GitLab URL, for example, `https://gitlab.example.com`. Required parameter | — |

After applying the dataset, fill in the "GitLab token" and "GitLab username" credentials in the user profile.

GitLab groups, repositories, and widgets for the Home page are created by the "GitLab: groups", "GitLab: repository activity", and "GitLab: widgets" datasets.
They are described in ["Initial setup"](../home/setup/#wizard-datasets).

### Kubernetes

This dataset is designed for connecting to the Kubernetes API.

The dataset creates the "Kubernetes" external service and the "Kubernetes token" credential type, a service account token for accessing the Kubernetes API.

Configuration parameters:

| Name                 | Description                                                               | Default value         |
|----------------------|---------------------------------------------------------------------------|-----------------------|
| "Kubernetes API URL" | Base URL of the Kubernetes API server, for example, `https://api.cluster.example.com`. Required parameter | — |

After applying the dataset, fill in the "Kubernetes token" credentials in the user profile.

### Vault

This dataset is designed for connecting to the HashiCorp Vault API.

The dataset creates the "Vault" external service and the "Vault token" credential type, a token for API requests.

Configuration:

The dataset configures a connection to the Vault API through an external service with authorization by the `X-Vault-Token` header.
For details on the authorization format, refer to ["External services"](../external-services/#vault).

{{< alert level="info" >}}
After applying the dataset, specify the Vault token in the "Vault token" credentials in the super administrator profile.
{{< /alert >}}

Configuration parameters:

| Name        | Description                     | Default value                 |
|-------------|---------------------------------|-------------------------------|
| "Vault URL" | Base URL of the HashiCorp Vault API, for example, `https://vault.example.com`. Required parameter | —   |

Applying:

1. Apply the Vault dataset.
1. Specify the URL of the Vault server.
1. Go to the super administrator profile.
1. Fill in the "Vault token" credentials.
1. Connect the "Vault" external service to the actions, widgets, or datasources that access the Vault API.

After that, requests through the external service use the token from the user's credentials.
For example, you can connect the external service to the ["Vault secrets"](../widgets/generic/vault/) widget to view secrets in KV v2.

### Deckhouse managed PostgreSQL

A dataset for deploying managed PostgreSQL in Deckhouse Platform.

The dataset prepares a resource, actions, and processes for managing the managed PostgreSQL lifecycle: creating an instance in the cluster through the Kubernetes external service and saving the connection details in Vault or Stronghold.

Configuration:

Before applying the dataset, configure the [Kubernetes](../external-services/#kubernetes) and [Vault](../external-services/#vault) (or Stronghold) external services.

{{< alert level="info" >}}
A detailed description of the dataset components is available with the "Description" button on the card in the "Datasets" section.
{{< /alert >}}

Configuration parameters:

| Name | Description | Default value |
|----------|----------|-----------------------|
| "Kubernetes external service" | Kubernetes cluster where managed PostgreSQL will be deployed. Required parameter | — |
| "Vault or Stronghold external service" | External service for writing connection details to KV v2. Required parameter | — |
| "Vault path prefix" | KV v2 path prefix for saving connection details (with `/data/`) | `services/data/` |
| "Create catalog resource" | Create the dataset's own parent resource. Disable it if processes are bound to the canonical shared resources | Enabled |
| "Create actions" | Create actions for creating and deleting the managed service | Enabled |
| "Create processes" | Create processes for creating and deleting the managed service | Enabled |
| "Create datasource" | Collect managed service instances from the cluster into the catalog: status, environment, class, and allocated resources. Only objects with DDP labels are synced | Enabled |
| "Catalog resource for processes" | If set, processes are bound to this resource instead of the dataset's own parent resource. For the canonical Home page model, specify the UUID of the "Shared resources" resource | — |

### Deckhouse managed Valkey

A dataset for deploying managed Valkey in Deckhouse Platform.

The dataset prepares a resource, actions, and processes for managing the managed Valkey lifecycle: creating an instance in the cluster through the Kubernetes external service and saving the connection details in Vault or Stronghold.

Configuration:

Before applying the dataset, configure the [Kubernetes](../external-services/#kubernetes) and [Vault](../external-services/#vault) (or Stronghold) external services.

{{< alert level="info" >}}
A detailed description of the dataset components is available with the "Description" button on the card in the "Datasets" section.
{{< /alert >}}

Configuration parameters:

| Name | Description | Default value |
|----------|----------|-----------------------|
| "Kubernetes external service" | Kubernetes cluster where managed Valkey will be deployed. Required parameter | — |
| "Vault or Stronghold external service" | External service for writing connection details to KV v2. Required parameter | — |
| "Vault path prefix" | KV v2 path prefix for saving connection details (with `/data/`) | `services/data/` |
| "Create catalog resource" | Create the dataset's own parent resource. Disable it if processes are bound to the canonical shared resources | Enabled |
| "Create actions" | Create actions for creating and deleting the managed service | Enabled |
| "Create processes" | Create processes for creating and deleting the managed service | Enabled |
| "Create datasource" | Collect managed service instances from the cluster into the catalog: status, environment, class, and allocated resources. Only objects with DDP labels are synced | Enabled |
| "Catalog resource for processes" | If set, processes are bound to this resource instead of the dataset's own parent resource. For the canonical Home page model, specify the UUID of the "Shared resources" resource | — |

### Deckhouse managed Kafka

A dataset for deploying managed Kafka in Deckhouse Platform.

The dataset prepares a resource, actions, and processes for managing the managed Kafka lifecycle: creating an instance in the cluster through the Kubernetes external service and saving the connection details in Vault or Stronghold.

Configuration:

Before applying the dataset, configure the [Kubernetes](../external-services/#kubernetes) and [Vault](../external-services/#vault) (or Stronghold) external services.

{{< alert level="info" >}}
A detailed description of the dataset components is available with the "Description" button on the card in the "Datasets" section.
{{< /alert >}}

Configuration parameters:

| Name | Description | Default value |
|----------|----------|-----------------------|
| "Kubernetes external service" | Kubernetes cluster where managed Kafka will be deployed. Required parameter | — |
| "Vault or Stronghold external service" | External service for writing connection details to KV v2. Required parameter | — |
| "Vault path prefix" | KV v2 path prefix for saving connection details (with `/data/`) | `services/data/` |
| "Create catalog resource" | Create the dataset's own parent resource. Disable it if processes are bound to the canonical shared resources | Enabled |
| "Create actions" | Create actions for creating and deleting the managed service | Enabled |
| "Create processes" | Create processes for creating and deleting the managed service | Enabled |
| "Create datasource" | Collect managed service instances from the cluster into the catalog: status, environment, class, and allocated resources. Only objects with DDP labels are synced | Enabled |
| "Catalog resource for processes" | If set, processes are bound to this resource instead of the dataset's own parent resource. For the canonical Home page model, specify the UUID of the "Shared resources" resource | — |
