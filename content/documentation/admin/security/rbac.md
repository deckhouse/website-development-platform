---
title: Role model
description: Roles, permissions, bindings, ownership, and access control in Deckhouse Development Platform.
weight: 20
---

The Deckhouse Development Platform (DDP) role model defines which actions users can perform in the platform and which objects they can access. The model controls access to the API and web interface at both the platform and individual object levels.

The role model is implemented in DDP Backend and uses a PostgreSQL database to store roles, permissions, and their relationships.

## Role model components

The role model consists of the following components:

* A permission corresponds to a specific action in DDP.
* A role combines a set of permissions.
* A role binding associates a role with users or teams.

Each DDP object has its own permissions, roles, and role bindings. Global roles, permissions, and role bindings allow operations at the platform level.

{{< alert level="info" >}}
Without the required permissions, a user cannot perform operations in the platform. The system checks permissions for every operation.
{{< /alert >}}

## Object types and permissions

### Global permissions

Global permissions apply across the platform and grant access to all objects of a specific type.

Resources:
- `create:resources` — create resources.
- `read:resources` — view resources.
- `update:resources` — edit resources.
- `update:resources-order` — change the order and grouping of resources in the catalog.
- `delete:resources` — delete resources.

Entities:
- `create:entities` — create entities.
- `read:entities` — view entities.
- `update:entities` — edit entities.
- `delete:entities` — delete entities.

Data sources:
- `create:datasources` — create data sources.
- `read:datasources` — view data sources.
- `update:datasources` — edit data sources.
- `sync:datasources` — synchronize data sources.
- `delete:datasources` — delete data sources.

Actions:
- `create:actions` — create actions.
- `read:actions` — view actions.
- `update:actions` — edit actions.
- `run:actions` — run actions.
- `delete:actions` — delete actions.

Automations:
- `create:automations` — create automations.
- `read:automations` — view automations.
- `update:automations` — edit automations.
- `delete:automations` — delete automations.

Workflows:
- `create:workflows` — create workflows.
- `read:workflows` — view workflows.
- `update:workflows` — edit workflows.
- `delete:workflows` — delete workflows.

Webhooks:
- `create:webhooks` — create webhooks.
- `read:webhooks` — view webhooks.
- `update:webhooks` — edit webhooks.
- `delete:webhooks` — delete webhooks.

Widgets:
- `create:widgets` — create widgets.
- `read:widgets` — view widgets.
- `update:widgets` — edit widgets.
- `run:widget-actions` — run widget actions.
- `delete:widgets` — delete widgets.

Dashboards:
- `create:dashboards` — create dashboards.
- `read:dashboards` — view dashboards.
- `update:dashboards` — edit dashboards.
- `delete:dashboards` — delete dashboards.

MCP:
- `read:mcp-servers` — view connected MCP servers.
- `edit:mcp-servers` — connect, edit, synchronize the catalog of, and delete MCP servers.
- `read:mcp-collections` — view MCP collections.
- `edit:mcp-collections` — create, edit, and delete MCP collections.
- `read:mcp-tools` — view custom MCP tools.
- `edit:mcp-tools` — create, edit, and delete custom MCP tools.

External services:
- `create:external-services` — create external services.
- `read:external-services` — view external services.
- `update:external-services` — edit external services.
- `delete:external-services` — delete external services.

Trusted certificates:
- `create:trusted-certificates` — add trusted certificates.
- `read:trusted-certificates` — view trusted certificates.
- `update:trusted-certificates` — edit trusted certificates.
- `delete:trusted-certificates` — delete trusted certificates.

Processes:
- `create:processes` — create processes.
- `read:processes` — view processes.
- `update:processes` — edit processes.
- `delete:processes` — delete processes.

Teams:
- `create:teams` — create teams.
- `update:teams` — edit teams.
- `delete:teams` — delete teams.
- `update:team-variables` — edit team variables.
- `edit:team-filter-rules` — configure group filtering rules during synchronization from Dex.

Icons:
- `create:icons` — create icons.
- `delete:icons` — delete icons.

User credential types:
- `edit:user-access-credentials-types` — create, edit, and delete credential types; configure the Vault integration, including viewing and changing the configuration and testing the connection.
- `rotate:encryption-key` — rotate the encryption key. For details, see [Encryption key rotation](../encryption-key-rotation/).

Users:
- `edit:users` — create, block, unblock, and delete users.
- `impersonate:users` — start and end user impersonation. For details, see [Impersonation](../impersonation/).

{{< alert level="info" >}}
The `update:team-variables` permission allows users to edit variables only for teams of which they are members. Even a super administrator cannot change variables for a team to which they do not belong.
{{< /alert >}}

#### Page view permissions

Permissions with the `view:` prefix control access to sections of the platform interface:

- `view:admin-page` — access the "Administration" section.
- `view:self-service-page` — access the "Self-Service" section.
- `view:ai-page` — access the "AI" section.

{{< alert level="info" >}}
Without the corresponding `view:` permission, a user cannot see a section in the navigation menu even if they have other permissions for objects in that section.
{{< /alert >}}

### Object-level permissions

Each object type has permissions that apply only to a specific object.

For resources:
- `read:resource` — view a specific resource.
- `update:resource` — edit a specific resource.
- `delete:resource` — delete a specific resource.
- `create:entities` — create resource entities.
- `read:entities` — view resource entities.
- `update:entities` — edit resource entities.
- `delete:entities` — delete resource entities.
- `run:actions` — run actions for resource entities.
- `control:processes` — control processes for resource entities.
- `edit:role-bindings` — edit role bindings for the resource.

For entities:
- `read:entity` — view a specific entity.
- `update:entity` — edit a specific entity.
- `delete:entity` — delete a specific entity.
- `run:actions` — run actions for the entity.
- `control:processes` — control processes for the entity.
- `edit:role-bindings` — edit role bindings for the entity.

#### MCP collections

For MCP collections:
- `use:mcp-collections` — call collection tools in the AI assistant and through the platform MCP server.
- `edit:role-bindings` — edit role bindings for the collection.

For other objects, including actions, automations, processes, webhooks, widgets, dashboards, and external services:
- `read:[object-type]` — view the object.
- `update:[object-type]` — edit the object.
- `delete:[object-type]` — delete the object.
- `edit:role-bindings` — edit role bindings for the object.

## Permission check hierarchy

The role model is hierarchical. When processing a request, DDP Backend checks access rights in a specific order. For example, to determine whether a user can edit a specific entity, the backend performs the following checks:

### 1. Super administrator

- Check whether the user is a super administrator. If so, stop further checks.

### 2. Global roles

- Read the user's group membership from the JWT.
- Find global roles for the user and their groups through global role bindings.
- Check permissions in the global roles. If any global role bound to the user or their team contains `update:entities`, the user can edit any entity in DDP, and no further checks are performed.

### 3. Resource roles

- Find resource roles for the user and their groups through role bindings for the specific resource.
- Check permissions in the resource roles. If any resource role bound to the user or their team contains `update:entities`, the user can edit any entity of that resource, and no further checks are performed.

### 4. Entity roles

- Find entity roles for the user and their groups through role bindings for the specific entity.
- Check permissions in the entity roles. If any entity role bound to the user or their team contains `update:entity`, the user can edit that entity.

### 5. Object ownership when ownerIsAdmin is enabled

- Check whether the user owns the object.
- Check whether the user belongs to the team that owns the object.
- If the user owns the object or belongs to its owner team, grant the user all administrator permissions for the object.

If the required permission is not found at any level, the action is denied.

{{< alert level="info" >}}
Ownership is checked only when `ownerIsAdmin` is enabled in the platform configuration. For details, see [Object ownership](#object-ownership).
{{< /alert >}}

## Object ownership

Object ownership allows users and teams to own specific system objects and automatically receive all administrator permissions for those objects.

An object owner is a user or team assigned as the owner of a specific object.

### Effect of ownership on access rights

When `ownerIsAdmin` is enabled, an object owner receives all administrator permissions for that object. The owner can:

- View, edit, and delete the object.
- Manage role bindings and access for other users.
- Perform any action available to an object administrator.

### OwnerAsAdmin option

The `ownerIsAdmin` option controls access rights for object owners.

- "Enabled (`true`)" — object owners receive full administrator permissions for their objects.
- "Disabled (`false`)" — object owners receive no automatic permissions; roles alone control access.

{{< alert level="info" >}}
A platform administrator must configure `ownerIsAdmin` in the configuration file. The option is disabled by default.
{{< /alert >}}

### Automatic owner assignment on creation

When a user creates an object, such as an action, data source, widget, or resource, the platform automatically assigns the current user as its owner. The owner can be reassigned, or the object can be created without an owner.

### Ownership examples

#### Resource ownership

When a user creates a resource, they automatically become its owner. If `ownerIsAdmin` is enabled, the user receives all administrator permissions for that resource, including:
- Managing resource entities.
- Configuring role bindings.
- Editing and deleting the resource.

#### Entity ownership

When a user creates an entity, they become its owner and can:
- View and edit the entity.
- Manage role bindings for the entity.
- Delete the entity.

#### Owner team

If a team owns an object and `ownerIsAdmin` is enabled, all team members receive administrator permissions for the object.

## Default role

The platform allows one global role to be set as the default. Its permissions apply to all authenticated users. Configure the default role under "Administration" → "Access control" by using the switch in the "Roles" table.

Only a global role can be set as the default.

## Teams

Teams group users and allow roles to be assigned collectively. A user can belong to multiple teams, and their permissions are combined from all teams of which they are a member.

{{< alert level="info" >}}
Teams and team membership are synchronized from the external authentication system, Dex. Teams cannot be managed through the DDP interface.
{{< /alert >}}

## Configuring roles and managing access

### Creating a role

1. Go to "Administration" → "Access control".
1. Select the "Roles" tab.
1. Click "Create role".
1. Complete the form:
    - Name — unique role name.
    - Description — role purpose.
    - Object type — select the scope of the role:
       - `Global` — global permissions.
       - `Resources` — resource-level permissions.
       - `Entities` — entity-level permissions.
       - `Actions` — action-level permissions.
       - Other object types.
    - Permissions — select the required permissions.
1. Click "Save".

### Assigning a role to users

1. Go to "Administration" → "Access control".
1. Select the "Role bindings" tab.
1. Click "Create role binding".
1. Complete the form:
    - Name — role binding name.
    - Description — role binding purpose.
    - Role — select the role.
    - Object — select a specific object if the role is not global.
    - Users — add the users to whom the role is assigned.
    - Teams — add the teams to which the role is assigned.
1. Click "Save".

### Configuring the default role

1. Go to "Administration" → "Access control".
1. Select the "Roles" tab.
1. Find the global role to set as the default.
1. Enable the "Default role" switch.
1. Confirm the action.

{{< alert level="info" >}}
Only a global role can be the default.
{{< /alert >}}

### Editing roles

1. Go to "Administration" → "Access control".
1. Select the "Roles" tab.
1. Find the required role and click "Edit".
1. Make the required changes:
    - Change the name or description.
    - Add or remove permissions.
1. Click "Save".

### Editing role bindings

1. Go to "Administration" → "Access control".
1. Select the "Role bindings" tab.
1. Find the required role binding and click "Edit".
1. Make the required changes:
    - Change the name or description.
    - Add or remove users.
    - Add or remove teams.
1. Click "Save".

### Deleting roles and bindings

1. Go to the corresponding section.
1. Find the required role or binding.
1. Click "Delete".
1. Confirm the deletion.

{{< alert level="info" >}}
Deleting a role also deletes all associated role bindings.
{{< /alert >}}

### Viewing permissions for a user or team

#### For a user

1. Go to "Administration" → "Users".
1. Open the target user's profile.
1. Select the "Permissions" tab.
1. Review:
    - Global permissions.
    - Object-level permissions.
    - Roles assigned to the user.
    - Teams of which the user is a member.

#### For a team

1. Go to "Administration" → "Teams".
1. Open the target team's profile.
1. Select the "Permissions" tab.
1. Review:
    - The team's global permissions.
    - Object-level permissions.
    - Roles assigned to the team.
    - Users who belong to the team.

{{< alert level="info" >}}
Team membership is synchronized from the external authentication system, Dex.
{{< /alert >}}

### Role presets

The platform provides role presets for common access scenarios.

#### Global presets

##### "Admin" preset

- Type: `Global`.
- Permissions: all global permissions.
- Purpose: system administrators.

##### "Viewer" preset

- Type: `Global`.
- Permissions:
  - `read:actions`, `read:automations`, `read:dashboards`.
  - `read:datasources`, `read:entities`, `read:external-services`.
  - `read:processes`, `read:resource-relations`, `read:resources`.
  - `read:seeds`, `read:system-alerts`, `read:trusted-certificates`, `read:webhooks`.
  - `read:widgets`, `read:workflows`.
  - `read:audit-logs`, `view:admin-page`, `view:self-service-page`.
- Purpose: users with read-only access.

##### "Developer" preset

- Type: `Global`.
- Permissions:
  - `read:actions`, `read:dashboards`, `read:external-services`.
  - `read:processes`, `read:widgets`, `read:workflows`.
  - `run:actions`, `run:widget-actions`, `control:processes`.
  - `update:team-variables`.
- Purpose: developers who need to view information and run actions but do not need to create, edit, or delete objects. Developers see only entities to which role bindings at the resource or entity level grant them access.

##### "Platform engineer" preset

- Type: `Global`.
- Permissions: full access to the catalog and the "Self-Service" page.
- Purpose: engineers who configure the platform, including processes, data sources, and dashboards.

#### Process presets

##### "Process admin" preset

- Type: `Processes`.
- Permissions: `delete:process`, `edit:role-bindings`, `read:process`, `update:process`.
- Purpose: process administrators.

##### "Process viewer" preset

- Type: `Processes`.
- Permissions: `read:process`.
- Purpose: users with permission to view processes.

#### Resource presets

##### "Resource admin" preset

- Type: `Resources`.
- Permissions: all resource permissions.
- Purpose: administrators of specific resources.

#### Using presets

1. Go to "Administration" → "Access control".
1. Select the "Roles" tab.
1. Click "Create role".
1. Select an object type.
1. Under "Presets", click the required preset.
1. Configure the role:
    - Change the name and description if necessary.
    - Add or remove permissions.
1. Click "Save".

### Role configuration examples

#### "System administrator" role

- Type: `Global`.
- Permissions: all global permissions. Use the "Admin" preset.
- Assignment: super administrators.

#### "Resource administrator" role

- Type: `Resources`.
- Permissions: all resource permissions. Use the "Resource admin" preset.
- Assignment: a specific resource and the "Administrators" team.

#### "Process operator" role

- Type: `Processes`.
- Permissions: `delete:process`, `edit:role-bindings`, `read:process`, `update:process`. Use the "Process admin" preset.
- Assignment: the "Process operators" team.

## Best practices

### Creating roles

1. Determine the access level: decide whether you need a global role or a role for a specific object.
1. Select the required permissions according to the principle of least privilege.
1. Create the role in the corresponding administration section.
1. Assign the role to users or teams through role bindings.

### Managing access

1. Use teams to manage access for groups.
1. Use default roles to grant basic permissions to all users.
1. Follow the hierarchy: use global roles for broad access and object roles for specific access.
1. Regularly review assigned roles and bindings.

### Security

1. Apply the principle of least privilege: grant only the required permissions.
1. Audit regularly: review assigned roles and their use.
1. Use teams instead of individual assignments to simplify management.
1. Document roles with descriptions that explain their purpose.
