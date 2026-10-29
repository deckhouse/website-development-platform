---
title: Entity card
description: Description, team, owner, and properties of an entity on its page.
weight: 70
---

The widget displays the main information about the entity on whose page it is placed.

## Configuration

The widget does not require additional configuration.

## Displayed data

The widget displays the same information as the entity page in the catalog:

- Entity description with Markdown support: A long description is collapsed to three lines and expands with the "Show more" button.
- "Team" and "Owner": The team and the user that own the entity.
- "Properties" block: Values of entity properties, except for hidden resource properties.

The "Properties" block works the same way as on the entity page.
Its heading shows how many properties are filled in, and the tooltip lists the empty properties.
Properties of the "JSON", "YAML", "Markdown", and "Object" types are displayed in a separate "Documents" row.
The clock icon next to a property name opens its change history, and the "As of a moment" button shows the values of all properties at the selected moment.

## Placement

The widget works only on an entity page.
Add it to a dashboard that is displayed on the entity page.
The ways to connect dashboards to a resource are described in the "Dashboards" section.

If the widget is opened outside an entity page, it displays the "The widget must be bound to an entity" message.
