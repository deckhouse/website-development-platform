---
title: Entity calendar
weight: 10
---

The widget displays entities of the selected resource in a calendar.

## Displayed data

- **Weekly calendar** — a seven-day grid for the current week.
- **Entities by date** — all entities whose selected date field matches a given day.
- **Entity information** — the name and description, if specified, of each entity.
- **Week navigation** — buttons for moving to the previous or next week.

## Configuration

| Name       | Required | Description                                                                                                                   | Default |
|------------|----------|-------------------------------------------------------------------------------------------------------------------------------|---------|
| Resource   | Yes      | Resource whose entities are displayed in the calendar                                                                         | —       |
| Date field | Yes      | Field containing the date used to display the entity in the calendar. It can be a system field (`createdAt`, `updatedAt`) or a `Date` parameter | —       |

## Notes

- The widget displays the current week, from Monday through Sunday, by default.
- Use the **Previous week** and **Next week** buttons to navigate between weeks.
- Each day displays its date in `DD.MM` format.
- Entities are displayed as cards that link to the corresponding entity page.
- Entities with an empty or zero date are automatically excluded.
