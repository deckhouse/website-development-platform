---
title: Mapping rules
weight: 30
description: Reference for DDP Home mapping sets — the set form, areas, resource, relation, and property roles, action slots, template actions, widget slots, and validation errors.
---

Mapping rules map catalog resources, relations, properties, processes, and widgets to Home page concepts.
The rules are stored in Home mapping sets; the Home page uses the default set.
How the Home page selects a set is described in [Overview](overview/#default-mapping-set).

## The "Mapping rules" page

Mapping sets are located in "Administration" → "Directories" → "Mapping rules".

The panel above the table contains:

- The "Show Home" toggle: Enables and disables the Home page for all users.
- The "Create" button: Opens the form for a new set.
- The "Search" field: Filters sets by name and ID.

The sets table displays the following columns:

| Column | Content |
| --- | --- |
| "Name" | Set name |
| "Identifier" | Set ID |
| "Default for Home" | Default set toggle. The toggle is unavailable for the current default set: to change the set, enable the toggle for another set |
| "Resources" | Number of mapped resource roles |
| "Relations" | Number of mapped relation roles |
| "Properties" | Number of mapped property roles |
| "Action slots" | Number of filled action slots |
| "Templates" | Number of templates with at least one bound process |
| "Widgets" | Number of filled widget slots |

To delete a set, enter its ID in the confirmation window. The default set cannot be deleted.

Permissions for working with sets:

- `create:home-mapping-sets`: Create sets.
- `update:home-mapping-sets`: Edit sets, select the default set, and use the "Show Home" toggle.
- `delete:home-mapping-sets`: Delete sets.

No permission is required to view sets, since every Home page reads them.
For selection lists to be populated in the set form, the user also needs permissions to view resources, resource relations, processes (`read:processes`), widgets (`read:widgets`), and templates (`read:template-packs`).

## Set form

The set form opens with the "Create" button or the edit button in a table row. The form contains two tabs.

On the "Main information" tab, fill in the following fields:

- "Name": From 3 to 64 characters.
- "Identifier": A stable set ID: from 3 to 64 characters, lowercase Latin letters, digits, and hyphens.
  Cannot be changed after creation.
- "Default for Home": Makes the set the default set.
  The first set created becomes the default set automatically.

On the "Catalog mapping" tab, rules are grouped by Home page areas.
Each area expands separately and contains the following blocks:

| Block | Purpose |
| --- | --- |
| "Catalog resource" | The resource the area is built on |
| "Catalog resources" | Several area resources, each with its own properties and relations. Used in the "Microservices" and "Shared resources" areas |
| "Relations" | Relations the Home page uses to traverse the area's entities |
| "Properties" | IDs of entity properties the Home page reads |
| "Actions" | Action slots of the area |
| "Template actions" | Processes bound to a microservice template. Only in the "Microservices" area |
| "Widgets" | Widget slots. Only in the "Microservices" area |

Each form field has a tooltip describing the role's purpose, its effect, and its place on the Home page.

The "Specification editor" button opens the entire set as a JSON document. The "Back to form" button returns you to the form.
Deleting a key in the specification text does not remove the value from the form. Clear the value in the form or replace it in the text.

### Selecting relations and properties

The list of relations and properties for a role is built from the resource of its parent role.
For example, the `has_deployment` relation is selected from the relations of the `microservice` role resource, and the `environment_domain` property from the properties of the `environment` role resource.

Until the parent role is mapped to a resource, the field is unavailable and the tooltip "Configure the mapping of the related resource role first to select items from its catalog" is displayed.

When a role's resource changes, the form clears the relations and properties that depend on that role.
For example, when the `system` role resource changes, the `contains_service` and `contains_shared` relations are cleared in all resources of the "Microservices" and "Shared resources" areas.
Clearing happens in the form before saving; save the set to apply the changes.

### Multiple resources per area

In the "Microservices" and "Shared resources" areas, a role can be mapped to several catalog resources, for example, microservices from different accounting systems.
Each resource has its own properties and relations.

To add a resource, click "Add resource" and fill in the resource card.
A relation whose parent is in another area, for example, `contains_service` from a system to a microservice, becomes available for selection after that area's resource is mapped.
When the resource in a card changes, its properties and relations are cleared.

## Mapping areas

Each mapping area belongs to one Home page section. The table lists the areas, sections, and their roles:

| Area | Home page section | Resource role | Relation roles | Property roles | Slots |
| --- | --- | --- | --- | --- | --- |
| "Systems" | "Systems" | `system` | `has_ingress`, `system_has_deployment`, `system_has_integration` | `system_domain`, `system_lifecycle` | — |
| "Microservices" | "Microservices" | `microservice` (multiple resources) | `contains_service`, `has_deployment` | `microservice_*` | `deploy_microservice`, `delete_microservice`; template actions; widget slots |
| "Environments" | "Environments" | `environment` | `environment_hosts_ingress` | `environment_type`, `environment_ttl`, `environment_marked_for_deletion`, `environment_domain` | `create_environment`, `delete_environment` |
| "Shared resources" | "Resource ordering" | `shared_resource` (multiple resources) | `contains_shared`, `shared_has_deployment` | `shared_environment`, `shared_kind`, `shared_provider` | `order_resource`, `connect_shared_resource`, `disconnect_shared_resource`, `delete_shared_resource` |
| "Deployments" | "System deployment" | `deployment` | `deployment_in_environment` | `deployment_*` | — |
| "Ingress" | "System deployment" | `ingress` | `ingress_routes_to` | `ingress_host`, `ingress_routes` | — |
| "Clusters" | "Clusters and networks" | `cluster` | `cluster_hosts_environment` | `cluster_region`, `cluster_provider` | — |
| "Guides" | "How the platform works" | `guide` | — | `guide_category`, `guide_video_url`, `guide_doc_url` | — |
| "Integrations" | "Integrations" | `integration` | `integration_targets_system` | `integration_criticality`, `integration_link_type` | — |
| "Version control namespaces" | "Create microservice" | `vcs_namespace` | — | `vcs_namespace_id` | — |

If the set contains keys that do not belong to known areas, the form displays them in the "Other" area.

The "Teams", "Diagrams", and "Resource consumption" sections do not have their own areas and use the roles of other areas.

## Resource roles

A resource role specifies which catalog resource's entities the Home page displays as objects of a certain kind.
The "Canonical resource" column shows the resource created by the "Canonical architecture (schema)" dataset.

| Role | Name in the form | Purpose | Canonical resource |
| --- | --- | --- | --- |
| `system` | "System" | Systems: domain tree, dependencies, composition, owners | "Systems" (`systems`) |
| `microservice` | "Microservice" | Microservices: list, card, deployment, upgrade from template | "Microservices" (`microservices`) |
| `environment` | "Environment" | Environments. An environment is a Kubernetes namespace | "Namespaces" (`kubernetes-namespaces`) |
| `shared_resource` | "Shared resource" | Shared resources: databases, queues, caches | "Shared resources" (`shared-resources`) |
| `deployment` | "Deployment" | Deployments of microservices and systems to environments | "Deployments" (`kubernetes-deployments`) |
| `ingress` | "Ingress" | Entry points on the deployment diagram | "Ingresses" (`kubernetes-ingresses`) |
| `cluster` | "Cluster" | Clusters that host environments | "Clusters" (`clusters`) |
| `guide` | "Guide" | Cards in the "How the platform works" section | "Help" (`guides`) |
| `integration` | "Integration" | Integrations between systems | "Integrations" (`integrations`) |
| `vcs_namespace` | "Version control namespace" | Groups in which microservice repositories are created | "Groups" (`gitlab-groups`) |

## Relation roles

A relation role specifies which catalog relation the Home page uses to navigate from one entity to another.

| Role | Name in the form | Direction | Canonical relation |
| --- | --- | --- | --- |
| `contains_service` | "Contains microservice" | System → microservice | "System — microservice" |
| `contains_shared` | "Contains shared resource" | System → shared resource | "System — shared resource" |
| `system_has_integration` | "System takes part in integration" | Client system → integration | "System — integration" |
| `integration_targets_system` | "Integration targets system" | Integration → server system | "Integration — system" |
| `has_ingress` | "Has ingress" | System → Ingress | "System — ingress" |
| `environment_hosts_ingress` | "Environment hosts ingress" | Environment → Ingress | "Namespace — ingress" |
| `ingress_routes_to` | "Ingress routes to" | Ingress → microservice | "Ingress — microservice" |
| `has_deployment` | "Has deployment" | Microservice → deployment | "Microservice — deployment" |
| `system_has_deployment` | "System has deployment" | System → deployment | "System — deployment" |
| `shared_has_deployment` | "Shared resource has deployment" | Shared resource → deployment | "Shared resource — deployment" |
| `deployment_in_environment` | "Deployment in environment" | Deployment → environment | "Deployment — environment" |
| `cluster_hosts_environment` | "Cluster hosts environment" | Cluster → environment | "Cluster — environment" |

Systems are related to each other only through integrations.
A client system is related to an integration by the `system_has_integration` role, and an integration is related to a server system by the `integration_targets_system` role.

If a relation role is not mapped, the Home page uses any relation between entities of the corresponding resources.
This lets you work with a partially filled set.

## Property roles

A property role contains the ID of an entity property.
The property is selected from the properties of the parent role's resource: for `system_*`, the `system` role resource; for `microservice_*`, the `microservice` role resource; and so on.

### Systems, integrations, clusters, and guides

Property roles for systems, integrations, clusters, and guides:

| Role | Name in the form | Where it is displayed | Canonical property |
| --- | --- | --- | --- |
| `system_domain` | "System domain" | The "Domains" tree in the "Systems" section, the "Domain" column in the "Microservices" section | `domain` |
| `system_lifecycle` | "System lifecycle" | "Lifecycle" on the system's "Overview" tab | `lifecycle` |
| `integration_criticality` | "Integration criticality" | Integration "Criticality", the "Critical links" card | `criticality` |
| `integration_link_type` | "Integration link type" | Integration "Link type" | `link_type` |
| `cluster_region` | "Cluster region" | Cluster card | `region` |
| `cluster_provider` | "Cluster provider" | Cluster card | `provider` |
| `guide_category` | "Guide category" | Guide grouping and filter | `category` |
| `guide_video_url` | "Guide video URL" | The "Watch video" button | `video_url` |
| `guide_doc_url` | "Guide documentation URL" | The "Open documentation" button | `doc_url` |

An integration is considered critical if the `integration_criticality` property value is `high`, case-insensitive.

### Microservices

Property roles for microservices.
The values of the `microservice_binding_*` roles are written by the process that creates a microservice from a template; [upgrading from a template](../templates/updates/) relies on them.

| Role | Name in the form | Purpose | Canonical property |
| --- | --- | --- | --- |
| `microservice_language` | "Microservice language" | The "Language" column, the "All languages" filter | `language` |
| `microservice_repository` | "Microservice repository" | Repository link; `.context.repository.url` for widgets | `repository` |
| `microservice_gitlab_project_id` | "GitLab project ID" | `.context.repository.id` for widgets | `gitlab_repository_id` |
| `microservice_binding_template_id` | "Microservice template ID" | The template the microservice was created from. Used to select template actions | `template_id` |
| `microservice_binding_template_version` | "Microservice template version" | Current template version in the repository | `template_version` |
| `microservice_binding_template_source` | "Microservice template source" | Name of the registry the template was obtained from | `template_source` |
| `microservice_binding_template_update_policy` | "Microservice template update policy" | `manual` or `review-auto` | `template_update_policy` |
| `microservice_binding_template_upgrade_strategy` | "Microservice template upgrade strategy" | Upgrade strategy set for the microservice | `template_upgrade_strategy` |
| `microservice_binding_template_upgrade_overwrite` | "Microservice upgrade overwrite paths" | List of paths to overwrite set for the microservice | `template_upgrade_overwrite` |
| `microservice_binding_template_upgrade_add_if_missing` | "Microservice upgrade add-if-missing paths" | List of paths to add set for the microservice | `template_upgrade_add_if_missing` |
| `microservice_vcs_provider` | "Microservice VCS provider" | Microservice repository provider | `vcs_provider` |
| `microservice_vcs_external_service_uuid` | "Microservice VCS external service" | The GitLab external service that hosts the repository | `vcs_external_service_uuid` |
| `microservice_default_branch` | "Microservice default branch" | The branch to which the merge request with the upgrade is created | `default_branch` |
| `microservice_open_mr_count` | "Open merge requests" | The "Open MRs" metric | `open_mr_count` |
| `microservice_last_pipeline_status` | "Last pipeline status" | The "Last pipeline" metric | `last_pipeline_status` |
| `microservice_last_pipeline_id` | "Last pipeline ID" | Pipeline number in the "Last pipeline" metric | `last_pipeline_id` |
| `microservice_last_pipeline_web_url` | "Last pipeline link" | Link to the pipeline | `last_pipeline_web_url` |
| `microservice_last_commit_sha` | "Last commit SHA" | The "Last delivery" line | `last_commit_sha` |
| `microservice_last_commit_message` | "Last commit message" | The "Last delivery" line | `last_commit_message` |
| `microservice_last_commit_author` | "Last commit author" | The "Last delivery" line | `last_commit_author` |

If the `microservice_binding_*`, `microservice_vcs_*`, or `microservice_default_branch` roles are not mapped, the upgrade-from-template mechanism uses the canonical property IDs from the table.

### Environments

Property roles for environments:

| Role | Name in the form | Purpose | Canonical property |
| --- | --- | --- | --- |
| `environment_type` | "Environment type" | The "Type" column; environment color on the deployment diagram | `type` |
| `environment_ttl` | "Environment TTL" | The "TTL" column. An environment with a TTL is considered temporary | `deletion_ttl` |
| `environment_marked_for_deletion` | "Environment marked for deletion" | The "Marked for deletion" tag when the value is `true`, `1`, or `yes` | `marked_for_deletion` |
| `environment_domain` | "Environment domain" | The "Domain" column. The value is changed from the environment card | `domain` |

An environment is also considered temporary if its type contains `temp` or `preview`, case-insensitive.

In the canonical model, the `deletion_ttl` property is filled in from the namespace's `ddp.deckhouse.io/deletion-ttl` label on every synchronization.
The `type`, `domain`, and `marked_for_deletion` properties are not synchronized from the cluster and are set in the catalog.

### Shared resources and deployments

Property roles for shared resources, deployments, Ingresses, and VCS namespaces:

| Role | Name in the form | Purpose | Canonical property |
| --- | --- | --- | --- |
| `shared_environment` | "Shared resource environment" | The environment that hosts the resource | `environment` |
| `shared_kind` | "Shared resource type" | Resource type, for example, PostgreSQL | `kind` |
| `shared_provider` | "Shared resource provider" | Provider, for example, Deckhouse | `provider` |
| `deployment_replicas` | "Deployment replicas" | Number of replicas | `replicas` |
| `deployment_cpu_request` | "Deployment CPU request" | CPU request | `cpu_request` |
| `deployment_memory_request` | "Deployment memory request" | Memory request | `memory_request` |
| `deployment_cpu_limit` | "Deployment CPU limit" | CPU limit | `cpu_limit` |
| `deployment_memory_limit` | "Deployment memory limit" | Memory limit | `memory_limit` |
| `deployment_image_tag` | "Deployment image tag" | Image version | `image_tag` |
| `ingress_host` | "Ingress host" | The "Host" line on the Ingress card | `host` |
| `ingress_routes` | "Ingress routes" | Routes: a list of objects with the `path` and `microservice` (microservice ID) fields | `routes` |
| `vcs_namespace_id` | "Version control namespace id" | ID of the group in which the repository is created | `id` |

CPU and memory values are specified in the Kubernetes format, for example, `200m`, `0.5`, `256Mi`, `1Gi`.

A shared resource is displayed on the environment diagram if the `shared_environment` property is empty or equals the environment ID.

## Action slots

An action slot contains processes that the Home page runs with a button or menu item.
Slots are filled in the area's "Actions" block.

| Slot | Name in the form | Section and button | Categories |
| --- | --- | --- | --- |
| `deploy_microservice` | "Deploy microservice to an environment (no template)" | "System deployment" → "Deploy microservice" for microservices without a template | No |
| `delete_microservice` | "Delete microservice from an environment (no template)" | "System deployment" → deleting a deployment of a microservice without a template | No |
| `create_environment` | "Create environment" | "Environments" → "Create environment" | No |
| `delete_environment` | "Delete environment" | "Environments" → environment card → "Delete environment" | No |
| `order_resource` | "Order resource" | "Resource ordering" and "System deployment" → "Order resource" | Resource type, for example, `PostgreSQL` |
| `connect_shared_resource` | "Connect shared resource" | "System deployment" → dragging a resource onto a deployment | No |
| `disconnect_shared_resource` | "Disconnect shared resource" | "System deployment" → dragging a resource onto a connected deployment | No |
| `delete_shared_resource` | "Delete shared resource" | "Resource ordering" and "System deployment" → deleting a resource | Resource provider and type, for example, `Deckhouse PostgreSQL` |

You can add several processes to a slot with the "Add process" button.
If a slot contains one process, it is selected automatically in the run window; if there are several, the user selects a process in the run window.

### Categories

Categories separate processes within a slot when different processes run for different resource types.
Categories are used only in the `order_resource` and `delete_shared_resource` slots; in other slots, the category field is not displayed.

To add a category, click "Add category" and specify a value in the "Resource category" field.
The `default` category contains processes without a category.
They are displayed first and are used when there is no exact match.

In the `order_resource` slot, the category is the type of the resource being ordered.
The Home page groups processes by category in the "Service" list and builds a type filter on the process cards.

In the `delete_shared_resource` slot, the deletion process is selected based on the properties of the specific resource in the following order:

1. The `<PROVIDER> <TYPE>` category, for example, `Deckhouse PostgreSQL`, based on the values of the `shared_provider` and `shared_kind` roles.
1. The `<TYPE>` category, for example, `PostgreSQL`.
1. The `default` category.

The first non-empty category is used. If there are no matching processes, the resource delete button is unavailable.

A category can contain Latin letters, digits, spaces, and the `_`, `.`, and `-` characters, must start with a letter or digit, and can be up to 64 characters long.

## Template actions

Template actions define processes for microservices created from a specific registry template.
The block is located in the "Microservices" area.

Each row of the block corresponds to one template. For a row, select a template in the "Template" field and the action processes:

| Action | Slot | Where it is run |
| --- | --- | --- |
| "Create microservice" | `create_microservice` | "Create microservice" → "Create microservice" |
| "Deploy microservice to an environment" | `deploy_microservice` | "System deployment" → "Deploy microservice" |
| "Delete microservice from an environment" | `delete_microservice` | "System deployment" → deleting a deployment |
| "Upgrade microservice from template" | `upgrade_service_from_template` | "Microservices" → microservice card → "Upgrade", as well as automatic upgrades |

One process is selected for each template action.
A template can be added to only one row.
The "Add template" button is unavailable while templates are loading, while any row has no template selected, or when no unused templates are left.

The "Template" list displays templates from the [template registry](../templates/registry/) in their latest available version.
If a template with bound processes is no longer available in the registry, the row is marked "not in the registry".
If the registry is empty, the tooltip "No templates available in the template registry — synchronize the registry to bind actions" is displayed.

The Home page determines a microservice's template by the `microservice_binding_template_id` property role in that microservice's resource and selects processes as follows.
If the microservice has a template, the process bound to that template is used.
If no process is bound for an action, the action is unavailable and shows a tooltip, for example, "No deploy process is set for the {template} template".
If the microservice has no template, deployment to and deletion from an environment use the processes of the `deploy_microservice` and `delete_microservice` slots from the "Actions" block.

A process bound to a different template is not used for the microservice, and processes from the "Actions" block do not replace a missing template binding.

The `create_microservice` and `upgrade_service_from_template` processes are set only in the "Template actions" block; the "Actions" block has no such slots.

## Widget slots

A widget slot displays an existing widget in a fixed place on the Home page screen.
Slots are filled in the "Widgets" block of the "Microservices" area.

| Slot | Name in the form | Where it is displayed |
| --- | --- | --- |
| `microservice_repository_widget` | "'Pipelines' tab widget" | "Microservices" → microservice card → "Pipelines" tab |
| `microservice_merge_requests_widget` | "'MRs' tab widget" | "Microservices" → microservice card → "MRs" tab |

Exactly one widget is selected for each slot. If a slot is not filled, the tab displays "This section is not available yet".

The Home page passes the selected microservice's context to the widget. In the widget configuration, the context is available through templating:

| Path | Value |
| --- | --- |
| `.context.microservice.uuid` | UUID of the microservice entity |
| `.context.microservice.slug` | Microservice ID |
| `.context.microservice.name` | Microservice name |
| `.context.repository.id` | Value of the `microservice_gitlab_project_id` role; an empty string if the value is not set |
| `.context.repository.url` | Value of the `microservice_repository` role |

For a configuration example, see [Widget on a microservice tab](../examples/home-widget-slot/).

## Configuring mapping from the Home page

You can change a section's mapping rules directly on the Home page without going to "Administration".
Changes are saved to the default mapping set.

To change a section's rules:

1. At the bottom of the Home page side menu, enable the "Configure mapping" toggle.
1. Click the settings icon next to the section name. The icon is displayed for sections that have mapping areas.
1. In the "Mapping: {section}" window, change the fields of the section's areas and click "Save".

After saving, Home page pages load data according to the new rules.

The "Configure mapping" toggle is available to users with the `update:home-mapping-sets` permission.
If there is no mapping set, the window displays the message "No active mapping set. Create one in Administration → Directories → Mapping rules.".

## Validation on save

Before saving, the form validates field values and checks that the selected objects exist in the catalog.

| Message | Cause |
| --- | --- |
| "Enter a name" | The "Name" field is empty |
| "Some mapping targets were deleted from the catalog. Update or clear them: {details}" | A resource, relation, property, process, or widget selected in the set has been deleted from the catalog. `{details}` lists the roles, for example, `resource "system"` |
| "Failed to load catalog data for mapping validation. Try again later." | Failed to load catalog objects for validation |
| "Fix the errors in the specification editor before saving" | The specification editor contains errors |
| "Invalid mapping document: {details}" | A value failed format validation when saving a section from the Home page. `{details}` contains the path to the value |
| "Invalid Home mapping document" | The set document failed portal validation |
| "Cannot unset the default home mapping set without assigning another default" | The "Default for Home" flag was disabled for the default set |
| "Cannot delete the default Home mapping set" | An attempt to delete the default set |

The portal validates the value format:

- Property ID: Lowercase Latin letters, digits, `_`, and `-`, up to 64 characters.
- Template ID: Lowercase Latin letters, digits, and the `.`, `_`, `/`, and `-` characters, up to 128 characters.
- Widget slot: Exactly one widget.
