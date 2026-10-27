---
title: Custom resource ordering process
weight: 32
description: Adding a custom process for ordering and deleting a shared resource to the DDP Home page — parameters from the run context, registering the resource in the catalog, and the order_resource and delete_shared_resource slot categories.
---

This guide describes adding ordering of a custom resource type to the Home page — a Redis cache provided by an internal resource ordering system through an HTTP API.
As a result, Redis is shown in the "Resource ordering" section as a separate resource type, is ordered for the selected system and environment, and is deleted from the "Recent resources" list.

## Before you begin

To follow this guide, prepare the following:

- Permissions to create actions and processes, and the `update:home-mapping-sets` permission.
- An [external service](../external-services/) for the resource ordering system API.
- A mapping set in which the `system`, `environment`, and `shared_resource` roles are mapped, for example, the set created by the [initial setup wizard](../home/setup/).

The example uses property identifiers of the canonical catalog model: the "Shared resources" resource with the `kind`, `provider`, and `environment` properties.
For a custom catalog model, use the properties mapped to the `shared_kind`, `shared_provider`, and `shared_environment` roles.

## Ordering action

Create an action that calls the ordering system API and registers the resource in the catalog:

1. Create an HTTP request action with the ordering system's external service.
   Creating an action is described in [Actions](../actions/overview/).
1. Set the request body with the process parameters:

   ```json
   {
     "name": "{{ .process.instance_name }}",
     "namespace": "{{ .process.environment }}",
     "owner": "{{ .process.system }}",
     "type": "redis"
   }
   ```

1. Configure [entity creation](../actions/overview/#entity-creation) in the "Shared resources" resource after the action completes successfully:
   - Name and identifier: `{{ .process.instance_name }}`
   - `kind` property: `Redis`
   - `provider` property: `Internal`
   - `environment` property: `{{ .process.environment }}`
1. Configure [relation creation](../actions/overview/#entity-relation-creation) between the created resource and the run's system, using the relation mapped to the `contains_shared` role.

The `kind` and `provider` values determine the category in which the Home page looks for the resource deletion process.

## Ordering process

The process gets the system and environment from the run context: the system selected in the "Resource ordering" section header and the environment selected in the "Order resource" dialog.

Create the process:

1. Create a process named "Order Redis" and add a task with the ordering action to it.
   Creating a process is described in [Processes](../processes/overview/).
1. In the "Resource" field, bind the ordering process to the resource mapped to the `system` role ("Systems" in the canonical model).
   The Home page runs the order on the selected system's entity.
1. Add the process parameters:

   | Identifier | Default value | Settings |
   | --- | --- | --- |
   | `environment` | `{{ .context.environment.slug }}` | Required |
   | `system` | `{{ .context.system.slug }}` | Required |
   | `instance_name` | `{{ .context.system.slug }}-{{ .context.environment.slug }}` | Required, show in simplified form |

1. Fill in the process description: it is shown on the process card in the "Resource ordering" section.

The default resource name `<SYSTEM_ID>-<ENVIRONMENT_ID>` matches the name of the temporary row that the Home page shows while the order is in progress.
If the user changes the name, the temporary row disappears when the process completes, and the created resource appears in the list after the data is reloaded.

## Deletion process

The deletion process runs for a shared resource entity, so the process gets the resource data from the `.entity` root.

Create the "Delete Redis" process:

1. Add a task with an action that calls the resource deletion API and deletes the entity from the catalog after it completes successfully.
   Entity deletion is configured in the [Deleting entities](../actions/overview/#entity-deletion) section of the action.
1. Bind the deletion process to the "Shared resources" resource or to the resource mapped to the `shared_resource` role in your model.
1. Add the required `instance_name` parameter with the default value `{{ .entity.slug }}` and use it in the request body.

## Adding processes to the Home page

Add the processes to the mapping set's action slots:

1. Go to "Administration" → "Directories" → "Mapping rules" and open the default set.
1. On the "Catalog mapping" tab, expand the "Shared resources" area and find the "Actions" block.
1. In the "Order resource" slot, click "Add category", specify the `Redis` category, and add the "Order Redis" process.
1. In the "Delete shared resource" slot, click "Add category", specify the `Internal Redis` category, and add the "Delete Redis" process.
1. Click "Save".

The `Internal Redis` category consists of the resource's `provider` and `kind` property values.
The Home page uses it to select the deletion process for Redis resources and does not offer this process for resources of other types.

## Verifying the result

Check ordering and deleting the resource:

1. Open the Home page → "Resource ordering" and select a system.
1. Make sure the "Resource types" indicator contains `Redis` and the process catalog contains the "Order Redis" card.
1. Click "Run" on the card, select an environment, and click "Next".
1. In the "Run process" dialog, check that the `instance_name` parameter is filled in with `<SYSTEM>-<ENVIRONMENT>`, and click "Run".
1. Wait until the row labeled "creating" in the "Recent resources" list is replaced by the created resource.
1. Click "Delete" in the resource row and run the deletion process.

If the `environment` parameter is not filled in, check that the default value template starts with `.context` and that an environment is selected in the "Order resource" dialog.
The context of each Home page scenario is described in [Home page processes](../home/processes/#entity-and-context-by-scenario).
