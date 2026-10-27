---
title: Role model
description: Roles, permissions, bindings, ownership, and access control in Deckhouse Development Portal.
weight: 20
---

The Deckhouse Development Portal (DDP, portal) role model defines which actions users can perform in the portal and which objects they can access.
The model controls access to the API and web interface at both the portal and individual object levels.

The role model is implemented in DDP Backend and uses a PostgreSQL database to store roles, permissions, and their relationships.

## Role model components

The role model consists of the following components:

- A permission corresponds to a specific action in DDP.
- A role combines a set of permissions.
- A role binding associates a role with users or teams.

Each DDP object has its own permissions, roles, and role bindings.
Global roles, permissions, and role bindings allow operations at the portal level.

{{< alert level="info" >}}
Without the required permissions, a user cannot perform operations in the portal.
The system checks permissions for every operation.
{{< /alert >}}

## Object types and permissions

Permissions in the role model are global or object-level.

### Global permissions

Global permissions apply across the portal and grant access to all objects of a specific type.

Resources:

- `create:resources`: Create resources.
- `read:resources`: View resources.
- `update:resources`: Edit resources.
- `update:resources-order`: Change the order and grouping of resources in the catalog.
- `delete:resources`: Delete resources.

Resource relations:

- `create:resource-relations`: Create relations between resources.
- `read:resource-relations`: View relations between resources.
- `update:resource-relations`: Edit relations between resources.
- `delete:resource-relations`: Delete relations between resources.

Entities:

- `create:entities`: Create entities.
- `read:entities`: View entities.
- `update:entities`: Edit entities.
- `delete:entities`: Delete entities.

Datasources:

- `create:datasources`: Create datasources.
- `read:datasources`: View datasources.
- `update:datasources`: Edit datasources.
- `sync:datasources`: Synchronize datasources.
- `delete:datasources`: Delete datasources.

Actions:

- `create:actions`: Create actions.
- `read:actions`: View actions.
- `update:actions`: Edit actions.
- `run:actions`: Run actions.
- `delete:actions`: Delete actions.

Automations:

- `create:automations`: Create automations.
- `read:automations`: View automations.
- `update:automations`: Edit automations.
- `delete:automations`: Delete automations.

Workflows:

- `create:workflows`: Create workflows.
- `read:workflows`: View workflows.
- `update:workflows`: Edit workflows.
- `delete:workflows`: Delete workflows.

Webhooks:

- `create:webhooks`: Create webhooks.
- `read:webhooks`: View webhooks.
- `update:webhooks`: Edit webhooks.
- `delete:webhooks`: Delete webhooks.

Widgets:

- `create:widgets`: Create widgets.
- `read:widgets`: View widgets.
- `update:widgets`: Edit widgets.
- `run:widget-actions`: Run widget actions.
- `delete:widgets`: Delete widgets.

Dashboards:

- `create:dashboards`: Create dashboards.
- `read:dashboards`: View dashboards.
- `update:dashboards`: Edit dashboards.
- `delete:dashboards`: Delete dashboards.
- `create:page-dashboards`: Add dashboards to the "Administration", "Catalog", and "Self-Service" pages.
- `delete:page-dashboards`: Remove dashboards from the "Administration", "Catalog", and "Self-Service" pages.

MCP:

- `read:mcp-servers`: View connected MCP servers.
- `edit:mcp-servers`: Connect, edit, synchronize the catalog of, and delete MCP servers.
- `read:mcp-collections`: View MCP collections.
- `edit:mcp-collections`: Create, edit, and delete MCP collections.
- `read:mcp-tools`: View custom MCP tools.
- `edit:mcp-tools`: Create, edit, and delete custom MCP tools.

External services:

- `create:external-services`: Create external services.
- `read:external-services`: View external services.
- `update:external-services`: Edit external services.
- `delete:external-services`: Delete external services.

Trusted certificates:

- `create:trusted-certificates`: Add trusted certificates.
- `read:trusted-certificates`: View trusted certificates.
- `update:trusted-certificates`: Edit trusted certificates.
- `delete:trusted-certificates`: Delete trusted certificates.

Processes:

- `create:processes`: Create processes.
- `read:processes`: View processes.
- `update:processes`: Edit processes.
- `delete:processes`: Delete processes.
- `control:processes`: Run, pause, resume, stop, retry, and delete process instances for entities.

Teams:

- `create:teams`: Create teams.
- `update:teams`: Edit teams.
- `delete:teams`: Delete teams.
- `update:team-variables`: Edit team variables.
- `edit:team-filter-rules`: Configure group filtering rules during synchronization from Dex.

Datasets:

- `read:seeds`: View datasets.
- `apply:seeds`: Apply datasets, run the initial setup wizard, and use the initial setup banner on the Home page.
- `delete:seeds`: Delete datasets.

Home page mapping rules:

- `create:home-mapping-sets`: Create Home page mapping sets.
- `update:home-mapping-sets`: Edit Home page mapping sets, select the default set, and enable or disable the Home page.
- `delete:home-mapping-sets`: Delete Home page mapping sets.

Microservice templates:

- `read:template-packs`: View template versions: the microservice creation gallery and new template version notifications.
- `edit:template-packs`: Deprecate, restore, and delete template versions.
- `read:template-registries`: View template registry connections, settings, and synchronization runs.
- `edit:template-registries`: Edit template registry connections and settings, and run synchronization.

Tags:

- `create:tags`: Create tags.
- `update:tags`: Edit tags.
- `delete:tags`: Delete tags.

System alerts:

- `create:system-alerts`: Create system alerts.
- `read:system-alerts`: View system alerts.
- `update:system-alerts`: Edit system alerts.
- `delete:system-alerts`: Delete system alerts.

Icon permissions are `create:icons` (create icons) and `delete:icons` (delete icons).

For user credential types, the `edit:user-access-credentials-types` permission is available.
It allows creating, editing, and deleting credential types and configuring the Vault integration, including viewing and changing the configuration and testing the connection.
The `rotate:encryption-key` permission allows rotating the encryption key.
For details, see [Encryption key rotation](../encryption-key-rotation/).

Users:

- `edit:users`: Create, block, unblock, and delete users.
- `create:users`: The permission is listed in the role form, but the portal does not check it.
  Creating users requires `edit:users`.
- `impersonate:users`: Start and end user impersonation. For details, see [Impersonation](../impersonation/).

Role and role binding permissions are `edit:roles` (create, edit, and delete roles)
and `edit:role-bindings` (create, edit, and delete role bindings for any object).

For audit logs, the `read:audit-logs` permission allows viewing audit logs and exporting them to CSV.
For details, see [Audit logs](../audit-logs/).

{{< alert level="info" >}}
The `update:team-variables` permission allows users to edit variables only for teams of which they are members.
Even a super administrator cannot change variables for a team to which they do not belong.
{{< /alert >}}

#### Page view permissions

Permissions with the `view:` prefix control access to sections of the portal interface:

- `view:admin-page`: Access the "Administration" section.
- `view:self-service-page`: Access the "Self-Service" section.
- `view:ai-page`: Access the "AI" section.

{{< alert level="info" >}}
Without the corresponding `view:` permission, a user cannot see a section in the navigation menu even if they have other permissions for objects in that section.
{{< /alert >}}

### Object-level permissions

Each object type has permissions that apply only to a specific object.

For resources:

- `read:resource`: View a specific resource.
- `update:resource`: Edit a specific resource.
- `delete:resource`: Delete a specific resource.
- `create:entities`: Create resource entities.
- `read:entities`: View resource entities.
- `update:entities`: Edit resource entities.
- `delete:entities`: Delete resource entities.
- `run:actions`: Run actions for resource entities.
- `control:processes`: Control processes for resource entities.
- `edit:role-bindings`: Edit role bindings for the resource.

For entities:

- `read:entity`: View a specific entity.
- `update:entity`: Edit a specific entity.
- `delete:entity`: Delete a specific entity.
- `run:actions`: Run actions for the entity.
- `control:processes`: Control processes for the entity.
- `edit:role-bindings`: Edit role bindings for the entity.

For actions, automations, datasources, workflows, webhooks, widgets, dashboards, and processes:

- `read:[object-type]`: View the object, for example, `read:action`.
- `update:[object-type]`: Edit the object.
- `delete:[object-type]`: Delete the object.
- `edit:role-bindings`: Edit role bindings for the object.

Some object types have additional permissions:

- `run:action`: Run a specific action.
- `sync:datasource`: Synchronize a specific datasource.
- `run:widget-action`: Run actions of a specific widget.

#### MCP collections

For MCP collections, the `use:mcp-collections` permission allows calling collection tools in the AI assistant and through the portal MCP server,
and the `edit:role-bindings` permission allows editing role bindings for the collection.

#### AI provider templates

For a specific template, the `read:ai-provider-template`, `edit:ai-provider-template`, `use:ai-provider-template`, and `edit:role-bindings` permissions are used.
The global `use:ai-provider-templates` permission allows using templates across the installation.

#### AI agents

For a specific connection, the `read:ai-agent`, `edit:ai-agent`, `use:ai-agent`, and `edit:role-bindings` permissions are used.
For an agent source, the `read:ai-agent-source`, `edit:ai-agent-source`, and `edit:role-bindings` permissions are used.

Agent usage is not limited to the owner.
Grant `use:ai-agent` to the required users or teams through a role binding.
Ownership and membership in the owner team grant permissions when the `owner-is-admin` setting is enabled.

#### Agent skills

For a skill, the `read:agent-skill`, `edit:agent-skill`, and `edit:role-bindings` permissions are used.
For a skill source, the `read:agent-skill-source`, `edit:agent-skill-source`, and `edit:role-bindings` permissions are used.

A skill assigned to an agent does not grant additional permissions.
Reading a skill during a run requires the `get_skill` tool, available through the MCP collections of the agent and of the user who started the run.

#### Workspaces and templates

Permissions can be granted on an individual object:

| Object | Permissions |
| --- | --- |
| Workspace | `read:workspace`, `update:workspace`, `cancel:workspace`, `delete:workspace`, `edit:role-bindings` |
| Template | `read:workspace-template`, `edit:workspace-template`, `use:workspace-template`, `edit:role-bindings` |
| Template source | `read:workspace-template-source`, `edit:workspace-template-source`, `edit:role-bindings` |

`update:workspace` allows starting, updating, and opening the IDE, and `cancel:workspace` allows stopping.
Retrying an operation also checks the permission for its type.
Creating a workspace always requires the global `create:workspaces` permission and the permission to use the selected template.

Attaching a workspace to a chat and running process jobs require the user's own workspace.
A process also requires a binding to its entity.

### Action run records

Permissions for an action run record are checked through the object it belongs to: the entity, its resource, or the action.
Super administrator and global permissions apply as usual.

| Operation | Process record | Action record for an entity | Action record without an entity |
| --- | --- | --- | --- |
| Viewing the record and its visibility in the list | Global or resource-level `read:entities`, `read:entity`, or entity ownership | Global or resource-level `read:entities`, `read:entity`, or entity ownership | Global `read:actions` or `read:action` on the action |
| Cancelling, restarting, and deleting | `control:processes` globally, on the resource, or on the entity | `run:actions` globally, on the resource, or on the entity | Global `run:actions` or `run:action` on the action |

The run initiator and approvers always see the record, for example, in the "My activities" panel, but can manage it only with the permissions from the table.
Only a user from the list of [approvers](../../actions/overview/#approval) can approve a run, and only the initiator can delete a temporary response.

Without the required permission, the record management buttons are disabled.

## Permission check hierarchy

The role model is hierarchical.
When processing a request, DDP Backend checks access rights in a specific order.
For example, to determine whether a user can edit a specific entity, the backend performs the following checks:

### 1. Super administrator

The portal checks whether the user is a super administrator. If so, no further checks are performed.

### 2. Global roles

At this step, the portal performs the following checks:

- Read the user's group membership from the JWT.
- Find global roles for the user and their groups through global role bindings.
- Check permissions in the global roles.
  If any global role bound to the user or their team contains `update:entities`, the user can edit any entity in DDP, and no further checks are performed.

### 3. Resource roles

The portal finds resource roles for the user and their groups through role bindings for the specific resource.
Then it checks permissions in the resource roles.
If any resource role bound to the user or their team contains `update:entities`, the user can edit any entity of that resource, and no further checks are performed.

### 4. Entity roles

The portal finds entity roles for the user and their groups through role bindings for the specific entity.
Then it checks permissions in the entity roles.
If any entity role bound to the user or their team contains `update:entity`, the user can edit that entity.

### 5. Object ownership when ownerIsAdmin is enabled

At this step, the portal performs the following checks:

- Check whether the user owns the object.
- Check whether the user belongs to the team that owns the object.
- If the user owns the object or belongs to its owner team, grant the user all administrator permissions for the object.

If the required permission is not found at any level, the action is denied.

{{< alert level="info" >}}
Ownership is checked only when `ownerIsAdmin` is enabled in the portal configuration.
For details, see [Object ownership](#object-ownership).
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
When it is "Enabled (`true`)", object owners receive full administrator permissions for their objects.
When it is "Disabled (`false`)", object owners receive no automatic permissions, and roles alone control access.

{{< alert level="info" >}}
A portal administrator must configure `ownerIsAdmin` in the configuration file. The option is disabled by default.
{{< /alert >}}

### Automatic owner assignment on creation

When a user creates an object, such as an action, datasource, widget, or resource, the portal automatically assigns the current user as its owner.
The owner can be reassigned, or the object can be created without an owner.

### Ownership examples

The following examples show how ownership affects access rights.

#### Resource ownership

When a user creates a resource, they automatically become its owner.
If `ownerIsAdmin` is enabled, the user receives all administrator permissions for that resource, including:

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

The portal allows one global role to be set as the default.
Its permissions apply to all authenticated users.
Configure the default role under "Administration" → "RBAC" by using the switch in the "Roles" table.

Only a global role can be set as the default.

## Teams

Teams group users and allow roles to be assigned collectively.
A user can belong to multiple teams, and their permissions are combined from all teams of which they are a member.

{{< alert level="info" >}}
Teams and team membership are synchronized from the external authentication system, Dex, each time a user signs in.
You can also create, edit, and delete teams in the web interface under "Administration" → "Teams" with the `create:teams`, `update:teams`, and `delete:teams` permissions.
Team membership changed manually is overwritten from Dex the next time the user signs in.
{{< /alert >}}

## Configuring roles and managing access

Roles and role bindings are configured in "Administration" → "RBAC".

### Creating a role

To create a role, follow these steps:

1. Go to "Administration" → "RBAC".
1. Select the "Roles" tab.
1. Click "Create role".
1. Complete the form:
    - Name: Unique role name.
    - Description: Role purpose.
    - Object type: Select the scope of the role:
       - `Global`: Global permissions.
       - `Resources`: Resource-level permissions.
       - `Entities`: Entity-level permissions.
       - `Actions`: Action-level permissions.
       - Other object types.
    - Permissions: Select the required permissions.
1. Click "Save".

### Assigning a role to users

To assign a role to users, follow these steps:

1. Go to "Administration" → "RBAC".
1. Select the "Role bindings" tab.
1. Click "Create role binding".
1. Complete the form:
    - Name: Role binding name.
    - Description: Role binding purpose.
    - Role: Select the role.
    - Object: Select a specific object if the role is not global.
    - Users: Add the users to whom the role is assigned.
    - Teams: Add the teams to which the role is assigned.
1. Click "Save".

### Configuring the default role

To set the default role, follow these steps:

1. Go to "Administration" → "RBAC".
1. Select the "Roles" tab.
1. Find the global role to set as the default.
1. Enable the "Default role" switch.
1. Confirm the action.

{{< alert level="info" >}}
Only a global role can be the default.
{{< /alert >}}

### Editing roles

To edit a role, follow these steps:

1. Go to "Administration" → "RBAC".
1. Select the "Roles" tab.
1. Find the required role and click "Edit".
1. Make the required changes:
    - Change the name or description.
    - Add or remove permissions.
1. Click "Save".

### Editing role bindings

To edit a role binding, follow these steps:

1. Go to "Administration" → "RBAC".
1. Select the "Role bindings" tab.
1. Find the required role binding and click "Edit".
1. Make the required changes:
    - Change the name or description.
    - Add or remove users.
    - Add or remove teams.
1. Click "Save".

### Deleting roles and bindings

To delete a role or binding, follow these steps:

1. Go to the corresponding section.
1. Find the required role or binding.
1. Click "Delete".
1. Confirm the deletion.

{{< alert level="info" >}}
Deleting a role also deletes all associated role bindings.
{{< /alert >}}

### Viewing permissions for a user or team

You can view the effective permissions of a user or a team.

#### For a user

To view user permissions, follow these steps:

1. Go to "Administration" → "Users".
1. In the target user's row, click "View user permissions".
1. In the "User permissions" dialog, review:
    - "Global permissions": The user's global permissions.
    - "Permissions for ...": Object-level permissions, one block per object.
      Each block shows the object type, the object name, and the role binding that grants the permissions.

   To filter the permissions, use the "Search" field.

#### For a team

To view team permissions, follow these steps:

1. Go to "Administration" → "Teams".
1. In the target team's row, click "View team permissions".
1. In the "Team permissions" dialog, review:
    - "Global permissions": The team's global permissions.
    - "Permissions for ...": Object-level permissions, one block per object.
      Each block shows the object type, the object name, and the role binding that grants the permissions.

   To filter the permissions, use the "Search" field.

{{< alert level="info" >}}
Team membership is synchronized from the external authentication system, Dex.
{{< /alert >}}

### Role configuration examples

The following examples show how to configure typical roles.

#### "System administrator" role

Configure the role as follows:

- Type: `Global`.
- Permissions: all global permissions. Use the "Admin" preset.
- Assignment: super administrators.

#### "Resource administrator" role

Configure the role as follows:

- Type: `Resources`.
- Permissions: all resource permissions. Use the "Resource admin" preset.
- Assignment: a specific resource and the "Administrators" team.

#### "Process operator" role

Configure the role as follows:

- Type: `Processes`.
- Permissions: `delete:process`, `edit:role-bindings`, `read:process`, `update:process`. Use the "Process admin" preset.
- Assignment: the "Process operators" team.

## Role presets

The portal provides role presets for common access scenarios.

### Global presets

The following presets are available for global roles.

#### "Admin" preset

The preset has the following settings:

- Type: `Global`.
- Permissions: all global permissions.
- Purpose: system administrators.

#### "Viewer" preset

The preset has the following settings:

- Type: `Global`.
- Permissions:
  - `read:actions`, `read:automations`, `read:dashboards`.
  - `read:datasources`, `read:entities`, `read:external-services`.
  - `read:processes`, `read:resource-relations`, `read:resources`.
  - `read:seeds`, `read:system-alerts`, `read:template-packs`, `read:trusted-certificates`, `read:webhooks`.
  - `read:widgets`, `read:workflows`.
  - `read:mcp-collections`, `read:mcp-servers`, `read:mcp-tools`.
  - `read:audit-logs`, `view:admin-page`, `view:ai-page`, `view:self-service-page`.
- Purpose: users with read-only access.

#### "Developer" preset

The preset has the following settings:

- Type: `Global`.
- Permissions:
  - `read:actions`, `read:dashboards`, `read:external-services`.
  - `read:processes`, `read:template-packs`, `read:widgets`, `read:workflows`.
  - `run:actions`, `run:widget-actions`, `control:processes`.
  - `update:team-variables`.
- Purpose: developers who need to view information and run actions but do not need to create, edit, or delete objects.
  Developers see only entities to which role bindings at the resource or entity level grant them access.

#### "Platform engineer" preset

The preset has the following settings:

- Type: `Global`.
- Permissions:
  - Catalog: all permissions for resources, resource relations, and entities, including `update:resources-order`.
  - "Self-Service" page: `view:self-service-page` and all permissions for actions, automations, dashboards, datasources, webhooks, workflows, processes, and widgets, including `run:actions`, `sync:datasources`, `control:processes`, `run:widget-actions`, `create:page-dashboards`, and `delete:page-dashboards`.
  - `update:team-variables`.
  - "AI" page: `view:ai-page`, `read:mcp-collections`, `edit:mcp-collections`, `read:mcp-servers`, `edit:mcp-servers`, `read:mcp-tools`, and `edit:mcp-tools`.
- Purpose: engineers who configure the portal, including processes, datasources, and dashboards.

### Process presets

The following presets are available for process-level roles.

#### "Process admin" preset

The preset has the following settings:

- Type: `Processes`.
- Permissions: `delete:process`, `edit:role-bindings`, `read:process`, `update:process`.
- Purpose: process administrators.

#### "Process viewer" preset

The preset has the following settings:

- Type: `Processes`.
- Permissions: `read:process`.
- Purpose: users with permission to view processes.

### Resource presets

The following preset is available for resource-level roles.

#### "Resource admin" preset

The preset has the following settings:

- Type: `Resources`.
- Permissions: all resource permissions.
- Purpose: administrators of specific resources.

### Using presets

To create a role from a preset, follow these steps:

1. Go to "Administration" → "RBAC".
1. Select the "Roles" tab.
1. Click "Create role".
1. Select an object type.
1. Under "Presets", click the required preset.
1. Configure the role:
    - Change the name and description if necessary.
    - Add or remove permissions.
1. Click "Save".

## Best practices

Follow these recommendations when configuring the role model.

### Creating roles

When creating roles, follow these recommendations:

1. Determine the access level: decide whether you need a global role or a role for a specific object.
1. Select the required permissions according to the principle of least privilege.
1. Create the role in the corresponding administration section.
1. Assign the role to users or teams through role bindings.

### Managing access

When managing access, follow these recommendations:

1. Use teams to manage access for groups.
1. Use default roles to grant basic permissions to all users.
1. Follow the hierarchy: use global roles for broad access and object roles for specific access.
1. Regularly review assigned roles and bindings.

### Security

To keep access secure, follow these recommendations:

1. Apply the principle of least privilege: grant only the required permissions.
1. Audit regularly: review assigned roles and their use.
1. Use teams instead of individual assignments to simplify management.
1. Document roles with descriptions that explain their purpose.
