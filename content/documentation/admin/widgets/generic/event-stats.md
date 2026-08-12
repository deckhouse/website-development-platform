---
title: Event statistics
description: Monitor entity events, Redis streams, and event trends with the Event statistics widget.
weight: 140
---

The widget displays statistics about events involving DDP entities. It contains three tabs:

1. **Event statistics** — A chart showing the number of events by type over the selected time range, with configurable time grouping.
1. **Top entities** — A table of entities that generated the most events.
1. **Redis events** — A table of Redis event streams. For each stream, it shows:
   - The stream name, which you can select to view all events.
   - The resource associated with the stream.
   - The number of events in the stream.
   - Information about the latest event: entity, resource, event type, and time.

## Query parameters

| Name         | Required | Description                                                                            | Default      |
| ------------ | -------- | -------------------------------------------------------------------------------------- | ------------ |
| Date from    | Yes      | Start date for selecting events                                                        | 3 days ago   |
| Date to      | Yes      | End date for selecting events                                                          | Current date |
| Interval     | No       | Chart grouping interval: seconds, minutes, hours, days, weeks, months, or years        | Hour         |
| Interval step | No      | Number of interval units used for grouping                                             | 1            |
| Top entities | No      | Number of entities with the most events to display in the table                        | 10           |

## Event types

The widget supports the following event types:

- `ENTITY_CREATED` — Entity created.
- `ENTITY_UPDATED` — Entity updated.
- `ENTITY_DELETED` — Entity deleted.

### Behavior

- The chart shows events over the selected time range with configurable time grouping. The default grouping is by hour.
- The table displays all events for each entity without date filtering.
- Deleted entities are shown using the names extracted from their event specifications.
- The **Redis events** tab lets you monitor events stored in Redis Streams:
  - Each stream shows its event count and latest event.
  - Selecting a stream name opens a dialog containing all events from that stream.
  - Streams are automatically associated with resources by the UUID in the stream name.
  - The stream view shows the latest 1,000 events, newest first. Older events are not displayed.
- Each table row contains information about the latest event for the entity.
- A detailed change history is available for each entity.
- Events for deleted resources are not displayed because they are removed from the database when the resource is deleted.
