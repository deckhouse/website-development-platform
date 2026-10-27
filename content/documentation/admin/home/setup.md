---
title: Initial setup
weight: 20
description: The DDP Home page initial setup wizard — presets, "Canonical architecture" datasets, their parameters, and the steps performed manually after the wizard.
---

The initial setup wizard applies prepared datasets that create the catalog model for the Home page, mapping rules, connections to Kubernetes, Vault, and GitLab, datasources, processes, and widgets.
After the wizard completes, the Home page displays data from clusters and repositories and lets you run processes.

## Before you begin

The wizard requires the `apply:seeds` permission.

Prepare the addresses of the external systems to be connected to the portal:

- Kubernetes API server URL, for example, `https://api.cluster.example.com`
- Vault or Deckhouse Stronghold URL, for example, `https://vault.example.com`
- GitLab URL, for example, `https://gitlab.example.com`

Datasets do not check whether they have been applied before.
Reapplying a dataset whose objects already exist in the portal fails with an error.
Run the wizard on a portal where the "Canonical architecture" datasets have not been applied yet.

## Running the wizard

The wizard is launched with the "Run" button in the "The platform is ready for setup" banner on the Home page
or with the "Initial setup" button in "Administration" → "Directories" → "Datasets".

Only users with the `apply:seeds` permission see the banner.
The banner is hidden for all users with the "Hide" button or automatically after all selected datasets are applied successfully.
To show the banner again, enable the "Show the initial setup banner" toggle in the "Datasets" section.

The Home page is disabled after the portal is installed, so on a new installation the banner is not displayed until the "Show Home" toggle is enabled.

## Wizard steps

The wizard consists of five steps: "Preset", "Select", "Parameters", "Review", and "Progress".

### Preset

At the "Preset" step, select one of the options.
The "Platform setup" preset creates the catalog schema, connections to Kubernetes, Vault, and GitLab, and the Home page processes and widgets.
The "Platform setup with demo data" preset also adds demo data: sample services, resources, and process runs.

You can delete the demo data later in the "Datasets" section.

### Selection

The "Select" step displays the preset's datasets grouped by layer: "Schema", "Demo data", "Infrastructure", "Providers / operations", and "Home wiring".
Clear the checkboxes for the datasets you do not need.

The wizard does not let you clear a dataset that other selected datasets depend on.
Such a dataset is marked with the "Required" tag and the "Required for: …" caption.
The "Canonical architecture (schema)" and "Canonical architecture (Home bindings)" datasets are locked while datasets with Home page processes or widgets are selected.

### Parameters

At the "Parameters" step, a panel with parameters is displayed for each selected dataset.
Panels with unfilled required parameters are marked with an asterisk.

The wizard fills in some parameters automatically.
External services and datasources created by datasets in the same run are substituted into the parameters of dependent datasets.
Fill in the external system URLs manually.
Dataset parameters are listed in [Wizard datasets](#wizard-datasets).

### Review

The "Review" step displays a table of datasets in the order they are applied, with the number of objects in each dataset.
Enable the "Confirm Application" toggle and click "Apply".

### Progress

At the "Progress" step, datasets are applied one by one.
Each dataset displays a status: "Pending", "Running", "Success", or "Error".

If a dataset fails to apply, the queue stops, and the objects of already applied datasets remain in the portal.
Fix the cause of the error and click "Retry from current" to continue from the dataset where the error occurred.
The "Back to review" button returns you to the "Review" step.

The wizard window cannot be closed while the queue is running.
After all datasets are applied successfully, the message "All selected datasets were applied successfully" is displayed.

## Wizard datasets

Datasets are applied in dependency order. The table lists the datasets included in the wizard presets:

| Dataset | What it creates | Parameters |
| --- | --- | --- |
| "Canonical architecture (schema)" | The "Architecture", "Systems", "Microservices", "Shared resources", "Kubernetes", "Ingresses", "Deployments", "Namespaces", "Secret bindings", "GitLab", "Groups", "Clusters", "Help", and "Integrations" catalog resources, the relations between them, and the "Home (default)" mapping set with the `home-default` ID | — |
| "Canonical architecture (demo data)" | Demo entities of the Billing system: systems, microservices, environments, shared resources, deployments, clusters, guides, and integrations. Included only in the "Platform setup with demo data" preset | — |
| "Kubernetes" | The "Kubernetes" external service and the "Kubernetes token" credential type | "Kubernetes API URL": Required |
| "Vault" | The "Vault" external service and the "Vault token" credential type | "Vault URL": Required |
| "GitLab" | The "GitLab" external service and the "GitLab token" and "GitLab username" credential types | "GitLab URL": Required |
| "Microservice template registry" | The "Create microservice from template", "Upgrade microservice from template", "Deploy microservice to an environment", and "Delete microservice from an environment" actions and processes | "VCS provider", "External service", "Deployments datasource", "Ingresses datasource" |
| "Kubernetes: deployments, ingresses, and namespaces" | Datasources that collect Deployments, Ingresses, and namespaces from the cluster | "Kubernetes external service" |
| "Shared resource binding" | The "Secret bindings" datasource and the "Connect secrets to application" and "Disconnect shared resource" processes | "Kubernetes external service", "Deployments datasource" |
| "Environment management" | The "Create environment" and "Delete environment" processes | "Kubernetes external service", "Namespaces datasource" |
| "GitLab: groups" | A GitLab groups datasource for selecting a namespace when creating a microservice | "GitLab external service" |
| "GitLab: repository activity" | A datasource that fills in the number of open merge requests, the last pipeline, and the last commit on the microservice card | "GitLab external service", "All repositories", "GitLab group IDs", "Include subgroups" |
| "GitLab: widgets" | The "GitLab: repository pipelines" and "GitLab: repository merge requests" widgets for the microservice card tabs | "GitLab external service" |
| "Deckhouse managed PostgreSQL", "Deckhouse managed Valkey", "Deckhouse managed Kafka" | Actions, processes for ordering and deleting the managed service, and a datasource | "Kubernetes external service", "Vault or Stronghold external service", "Vault path prefix", and object creation parameters |
| "Canonical architecture (Home bindings)" | Adds the processes and widgets of the selected datasets to the slots of the `home-default` mapping set. Applied only by the wizard | — |

If the "All repositories" parameter is disabled in the "GitLab: repository activity" dataset, fill in "GitLab group IDs".

Kubernetes datasources collect only objects with the `ddp.deckhouse.io/environment` label and skip the `d8-*` and `kube-*` namespaces.

The table shows which slots the "Canonical architecture (Home bindings)" dataset fills in:

| Dataset | Action slot | Category |
| --- | --- | --- |
| "Environment management" | `create_environment`, `delete_environment` | `default` |
| "Shared resource binding" | `connect_shared_resource`, `disconnect_shared_resource` | `default` |
| "Deckhouse managed PostgreSQL" | `order_resource`, `delete_shared_resource` | `PostgreSQL`, `Deckhouse PostgreSQL` |
| "Deckhouse managed Valkey" | `order_resource`, `delete_shared_resource` | `Valkey`, `Deckhouse Valkey` |
| "Deckhouse managed Kafka" | `order_resource`, `delete_shared_resource` | `Kafka`, `Deckhouse Kafka` |
| "GitLab: widgets" | Widget slots `microservice_repository_widget`, `microservice_merge_requests_widget` | — |

The "Microservice template registry" dataset does not add processes to the mapping rules.
They are bound to templates manually after the template registry is connected.

## Post-wizard steps

The wizard does not perform some settings that depend on the specific installation. Perform them manually:

1. Enable the Home page: "Administration" → "Directories" → "Mapping rules" → the "Show Home" toggle.
1. Fill in the credentials in the user profile: the Kubernetes service account token, the Vault token, and the GitLab personal access token and username.
   Home page processes perform actions with the credentials of the user who runs them, so every user who runs processes needs credentials.
1. Connect the template registry and synchronize it.
   The procedure is described in [Template registry](../templates/registry/).
1. Bind processes to templates: "Mapping rules" → the default set → "Catalog mapping" → "Microservices" → "Template actions".
   For each template, select the action processes: the "Create microservice from template" process for the "Create microservice" action, and the processes with the same names for the "Deploy microservice to an environment", "Delete microservice from an environment", and "Upgrade microservice from template" actions.
1. If the catalog contains microservices created without a template, fill in the "Deploy microservice to an environment (no template)" and "Delete microservice from an environment (no template)" slots in "Microservices" → "Actions".
1. In the catalog, fill in the environment properties that do not come from the cluster: type, cluster, domain, and the "marked for deletion" flag.
   The environment domain can also be changed from the "Environments" section on the Home page.

## Deleting a dataset

To delete a dataset, click "Delete" on its card in the "Datasets" section.

When you delete a dataset with Home page processes, only the processes and widgets of that dataset are removed from the mapping set slots.
The default mapping set is not deleted when a dataset is deleted.

For general rules for working with datasets, see [Datasets](../datasets/).
