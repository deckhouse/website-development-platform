---
title: Kaiten. Space cards
description: Configuration and displayed task details for the Kaiten space cards widget.
weight: 10
---

The widget displays the task structure of a Kaiten space as a multi-level **Board → Cards** table.
Use it to view tasks at every level of the work hierarchy and review key card details,
including status, urgency, blocking state, and assignees.

## Configuration

| Name     | Required | Description                        | Default |
|----------|----------|------------------------------------|---------|
| Space ID | Yes      | Kaiten space identifier            | —       |

## Query parameters

| Name          | Required | Description                    | Default     |
|---------------|----------|--------------------------------|-------------|
| My tasks      | No       | Filters by the current user    | `false`     |
| Created after | Yes      | Start date of the query period | 1 month ago |
| Created before | Yes     | End date of the query period   | Now         |

## Displayed data

Each card contains:

- Card name.
- Column (board status).
- Status (queued, in progress, or completed).
- Lane.
- Owner (avatar, name, and email).
- Participants.
- Due date and urgency.
- Blocking state.

## Authorization

Authorization is described in [External services](../../external-services/#kaiten).
