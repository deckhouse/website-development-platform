---
title: Entity calendar
description: Configuration and behavior of the widget that displays resource entities in a calendar.
weight: 10
---

The widget displays entities of the selected resource in a calendar.

## Configuration

When adding the widget, specify the following parameters:

| Name       | Required | Description                                                                                                                   | Default value |
|------------|----------|-------------------------------------------------------------------------------------------------------------------------------|---------|
| Resource   | Yes      | Resource whose entities are displayed in the calendar                                                                         | —       |
| Date field | Yes      | Field containing the date used to display the entity in the calendar. It can be a system field (`createdAt`, `updatedAt`) or a `Date` property | —       |

## Displayed data

The widget displays the following data:

- **Weekly calendar**: A seven-day grid for the current week.
- **Entities by date**: All entities whose selected date field matches a given day.
- **Entity information**: The name and description, if specified, of each entity.
- **Week navigation**: Buttons for moving to the previous or next week.

## Notes

The widget has the following specifics:

- The widget displays the current week, from Monday through Sunday, by default.
- Use the **Previous week** and **Next week** buttons to navigate between weeks.
- Each day displays its date in `DD.MM` format.
- Entities are displayed as cards that link to the corresponding entity page.
- Entities with an empty or zero date are automatically excluded.
