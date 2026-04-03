---
title: Catalog
weight: 30
---

The catalog is a section of the platform that contains a list of all resources, their parameters and relationships, and allows users to run scripts for resource entities.

The catalog can be populated manually or automatically using data sources or webhooks.

## Resources

A resource is a template or entity type in the platform catalog. A resource defines the data structure, parameters, and properties that all entities of a given type will inherit. For example, the "GitLab Repositories" resource describes the data structure for GitLab repositories, and each specific entity of this resource represents a separate repository.

The fundamental principle of the platform is that the data model is configured by the platform administrator. The administrator can independently create the resources they need.

### Resource Naming

When naming resources, follow these rules:

- **Resource Name** can be any name and does not require special prefixes or separators. - The **Resource ID** must be unique across the entire platform.

### Resource Grouping

Resources in the catalog can be organized hierarchically, allowing for logical grouping of related resources and improving catalog navigation.

To create a resource group, you need to:

1. **Create Resources** — create all the necessary resources that will be included in the group. For example, the "Gitlab" resource could be the parent of "Repositories," "Groups," and other resources.

2. **Link Child Resources to Parent Resources** — in the catalog sidebar, drag the child resource onto the parent resource. Child resources will be automatically linked to the parent and appear in the interface as nested elements.

3. **Configure Display** — in the catalog sidebar, parent resources are displayed with an expand/collapse icon, allowing you to show or hide child elements.

{{< alert level="info" >}}
Changing resource grouping (dragging and dropping resources in the catalog sidebar) requires the global `update:resources-order` permission. For more information on access rights, see the ["Role Model"](../../admin/security/rbac/).
{{< /alert >}}

{{< alert level="info" >}}
To view a child resource, the user must have `read:resources` permissions for the entire hierarchy of parent resources. For more information on access rights, see the ["Role Model"](../../admin/security/rbac/).
{{< /alert >}}

#### Displaying the Full Path

The platform interface (resource selectors, tags, relationship graphs) displays the full resource path with the chain of parent resources separated by the `/` character. For example, if the "Repositories" resource is a child of the "Gitlab" resource, the interface will display "Gitlab / Repositories."

### Resource Card

By default, after creating a resource, its card will display a table of entities for that resource. Clicking on an entity in the table takes you to its card.

If a dashboard is selected in the "Use Dashboard" field, it will be displayed on the resource card instead of the entity table.

### Relationships between Resources

Each resource can be linked to another resource or to itself via a two-way relationship. Relationships are configured by the platform administrator. After setting up relationships for a resource, each entity in that resource can be linked to one or more entities in another resource.

You can create both vertical relationships (an entity from one resource is linked to an entity from another resource) and horizontal relationships (entities within a single resource are linked).

## Entities

An entity is a specific unit of each resource. For example, if a resource is called "Groups" and is a child of the resource "Gitlab," then each Gitlab group registered in the platform is an entity of that resource.

Entities inherit resource parameters, but parameters can be specified separately for each entity.

### Naming Entities

When naming entities, follow these rules:

- **The entity name** can be any name and does not require special prefixes or separators.
- **The entity ID** must be unique within the resource.

### Entity Card

The entity card consists of built-in panels and custom panels. Built-in panels:

- **Overview** — contains blocks indicating the entity's owner, description, parameters, and relationships.
- **Action Triggers** — contains a list of actions that have been triggered for this entity.
- **Process Starts** — Contains a list of processes that have started for this entity.
- **Events** — Contains a list of events generated for this entity.

New dashboards can be added to each entity card.

Adding dashboards to one resource entity applies to all entities within that resource.

### Relationships between entities

Relationships for each entity are displayed in the entity card as a table and a graph. Each entity can be linked to one or more entities of the same or different resource, depending on the configured relationships between resources.

Relationships between entities can be created manually or automatically using data sources or webhooks.

### Events

When the specification of any entity is created, deleted, or modified, an event of the corresponding type is generated: ENTITY_CREATED, ENTITY_DELETED, or ENTITY_UPDATED. These events serve the following purposes:

- **Audit** — recording changes occurring to an entity during its lifecycle.
- **Configure automated reactions** — configure reactions to changes in the entity specification.

The event storage time is limited and can be configured through the platform configuration file.

### Exporting Entities to CSV

The platform provides the ability to export entities to a CSV file for subsequent data analysis and processing.

#### Exporting Entities for a Single Resource

To export entities for a specific resource:

1. Open the resource card in the catalog.
2. In the entity table, click the **Download .csv** button.
3. The exported file will include all resource entities, taking into account the applied filters, sorting, and pagination.

#### Exporting Entities from Multiple Resources

To export entities from multiple resources simultaneously:

1. In the catalog sidebar, click the **Download Entities** button.
2. In the dialog box that opens, select one or more resources.
3. Click the **Download .csv** button.
4. All entities for the selected resources will be combined into a single CSV file.

{{< alert level="info" >}}
Unloading entities requires the global `read:entities` permission. For more information on access rights, see the [Role Model](../../admin/security/rbac/) section.
{{< /alert >}}
