---
title: Entity Kanban board
weight: 20
---

The widget displays entities of the selected resource on a Kanban board.

## Displayed data

- **Columns** — configured board columns and a **No status** column for entities without a matching state parameter value.
- **Entity cards** — each entity displays its name, description if specified, check status, owner, and update date.
- **Moving cards** — dragging a card between columns updates the entity's state parameter value. This requires permission to modify entities.

## Configuration

| Name            | Required | Description                                                                                                                                                                                                                                                             | Default |
|-----------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| Resource        | Yes      | Resource whose entities are displayed on the board                                                                                                                                                                                                                      | —       |
| State parameter | Yes      | Entity parameter that determines the card column. Supported types: `String`, `Number`, `Boolean`, `Enum`, `List`, `Date`, `Percentage`, and `URL`                                                                                                                        | —       |
| Columns         | Yes      | List of board columns. For each column, specify a name (the board heading), value (the state parameter value; for `Enum` and `List` parameters, select from the available options), and color (the tag color in the heading). Drag columns to change their order | —       |

## Notes

- Entities without a matching state parameter value are displayed in the **No status** column.
- Cards cannot be moved without permission to modify entities.
