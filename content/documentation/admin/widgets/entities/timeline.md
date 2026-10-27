---
title: Entity timeline
description: Configuration and behavior of the timeline that displays resource entities by date.
weight: 40
---

The widget displays entities of the selected resource on a timeline.

## Configuration

When adding the widget, specify the following parameters:

| Name           | Required | Description                                                                                                         | Default value |
|----------------|----------|---------------------------------------------------------------------------------------------------------------------|---------|
| Resource       | Yes      | Resource whose entities are displayed on the timeline                                                              | —       |
| Start date field | Yes    | Field containing the period start date. It can be a system field (`createdAt`, `updatedAt`) or a `Date` property  | —       |
| End date field | Yes      | Field containing the period end date. It can be a system field (`createdAt`, `updatedAt`) or a `Date` property    | —       |

## Displayed data

The widget displays the following data:

- **Timeline chart**: A horizontal chart where each entity is represented by a bar spanning from its start date to its end date.
- **Entity information**: Hovering over a bar displays the entity name and the period's start and end dates.
- **Sorting**: Entities are sorted from oldest at the top to newest at the bottom.

## Notes

The widget automatically scales the timeline to display all entities.
Entities with invalid dates, where the start date is later than the end date, are automatically excluded.
