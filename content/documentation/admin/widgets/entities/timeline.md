---
title: Entity timeline
weight: 40
---

The widget displays entities of the selected resource on a timeline.

## Displayed data

- **Timeline chart** — a horizontal chart where each entity is represented by a bar spanning from its start date to its end date.
- **Entity information** — hovering over a bar displays the entity name and the period's start and end dates.
- **Sorting** — entities are sorted from oldest at the top to newest at the bottom.

## Configuration

| Name           | Required | Description                                                                                                         | Default |
|----------------|----------|---------------------------------------------------------------------------------------------------------------------|---------|
| Resource       | Yes      | Resource whose entities are displayed on the timeline                                                              | —       |
| Start date field | Yes    | Field containing the period start date. It can be a system field (`createdAt`, `updatedAt`) or a `Date` parameter  | —       |
| End date field | Yes      | Field containing the period end date. It can be a system field (`createdAt`, `updatedAt`) or a `Date` parameter    | —       |

## Notes

- The widget automatically scales the timeline to display all entities.
- Entities with invalid dates, where the start date is later than the end date, are automatically excluded.
