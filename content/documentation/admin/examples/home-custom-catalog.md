---
title: Home page for a custom catalog model
weight: 31
description: Configuring DDP Home page mapping rules for an existing catalog model — matching resources, relations, and properties to Home page roles without the "Canonical architecture" dataset.
---

This guide describes connecting the Home page to a catalog that already has its own resources for systems, microservices, and environments.
The "Canonical architecture" dataset is not applied in this scenario because the Home page uses the existing resources through mapping rules.

The example uses a catalog with the following resources:

- "Products": Systems with the "Domain" (`domain`) and "Status" (`status`) properties.
- "Services": Microservices with the "Language" (`lang`) and "Repository" (`repo_url`) properties.
- "Stands": Environments with the "Stand type" (`stand_type`) and "Domain" (`base_domain`) properties.
- "Workloads": Deployments with the `replicas` and `image` properties.

The resources are linked by the "Product — service", "Service — workload", and "Workload — stand" relations.

## Before you begin

To follow this guide, you need the `create:home-mapping-sets` and `update:home-mapping-sets` permissions, as well as permissions to view resources, resource relations, processes, and widgets so that the selection lists in the form are populated.

## Matching resources to roles

Create a table that matches the catalog's resources, relations, and properties to Home page roles. For example:

| Role | Catalog object |
| --- | --- |
| `system` | "Products" resource |
| `system_domain` | `domain` property of the "Products" resource |
| `system_lifecycle` | `status` property of the "Products" resource |
| `microservice` | "Services" resource |
| `microservice_language` | `lang` property of the "Services" resource |
| `microservice_repository` | `repo_url` property of the "Services" resource |
| `contains_service` | "Product — service" relation |
| `deployment` | "Workloads" resource |
| `deployment_replicas` | `replicas` property of the "Workloads" resource |
| `deployment_image_tag` | `image` property of the "Workloads" resource |
| `has_deployment` | "Service — workload" relation |
| `deployment_in_environment` | "Workload — stand" relation |
| `environment` | "Stands" resource |
| `environment_type` | `stand_type` property of the "Stands" resource |
| `environment_domain` | `base_domain` property of the "Stands" resource |

Leave roles that have no match in the catalog empty.
Sections whose main role is not mapped show the message "Configure mapping rules or apply the 'Canonical architecture' dataset".
For the full list of roles, see [Mapping rules](../home/mapping/#resource-roles).

## Creating a mapping set

Create a mapping set and make it the default set:

1. Go to "Administration" → "Directories" → "Mapping rules" and click "Create".
1. On the "Main information" tab, fill in the fields:
   - "Name": For example, `Home for products`.
   - "Identifier": For example, `home-products`.
   - Enable "Default for Home".
1. Go to the "Catalog mapping" tab.

## Filling in the areas

The areas are filled in one by one.
A role's relations and properties are selected from the resource of its parent role, so map the resources first.

Fill in the "Systems" area:

1. In the "Catalog resource" block, select "Products".
1. In the "Properties" block, select `domain` for "System domain" and `status` for "System lifecycle".

Fill in the "Microservices" area:

1. In the "Catalog resources" block, click "Add resource" and select "Services".
1. On the resource card, select the `lang` property for "Microservice language" and `repo_url` for "Microservice repository".
1. Select the "Product — service" relation for "Contains microservice" and the "Service — workload" relation for "Has deployment".

Fill in the "Environments" and "Deployments" areas:

1. In the "Environments" area, select the "Stands" resource and the `stand_type` property for "Environment type" and `base_domain` for "Environment domain".
1. In the "Deployments" area, select the "Workloads" resource, the "Workload — stand" relation for "Deployment in environment", and the `replicas` and `image` properties.

Click "Save".

If you change a role's resource after saving, the form clears the dependent relations and properties.
Check them before saving again.

## Verifying the result

Open the Home page and check the sections:

- "Systems": The "Domains" tree shows products grouped by the `domain` property.
  Products without a `domain` value are not shown in the tree.
- "Microservices": The table shows the services, their languages, and their products in the "System" column.
- "Environments": The table shows the stands with their types and domains.
- "Resource consumption": The table shows entities of the "Workloads" resource with the number of replicas.

If a section shows "Configure mapping rules or apply the 'Canonical architecture' dataset", check that the section's main role is mapped to a resource and that the user has permission to view that resource's entities.

## Processes

To run processes from the Home page, add processes to the action slots.
For example, to create stands with the "Create environment" button, follow these steps:

1. Open the mapping set, the "Environments" area, the "Actions" block.
1. In the "Create environment" slot, click "Add process" and select the stand creation process.
1. Save the set.

The requirements for Home page processes and the context they receive are described in [Home page processes](../home/processes/).
