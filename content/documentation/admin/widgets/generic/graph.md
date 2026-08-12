---
title: Graph
description: Configure charts that aggregate and visualize Deckhouse Development Platform object data.
weight: 130
---

The widget displays information about DDP objects using one of the following chart types:

* Bar chart.
* Doughnut chart.
* Pie chart.
* Polar area chart.
* Radar chart.

## Configuration

| Name                   | Required | Description                                                                         | Default |
| ---------------------- | -------- | ----------------------------------------------------------------------------------- | ------- |
| Chart type             | Yes      | Chart visualization type                                                            | —       |
| Table name             | Yes      | Database table containing the records to visualize                                  | —       |
| Field name             | Yes      | Field used to aggregate records                                                      | —       |
| Filters                | No       | Fields and values used to filter the retrieved records                              | —       |
| Aggregation type       | Yes      | Method used to group the retrieved records                                           | —       |
| Aggregation parameters | No       | Time range and grouping step used when aggregating records by date                   | —       |

When configuring the widget, account for differences between database field names and object specification field names. When structures are stored in the database, camelCase names from object specifications are converted to snake_case. For example:

* Specify the `createdAt` field as `created_at` in the widget configuration.
* Specify the `resourceUuid` field as `resource_uuid` in the widget configuration.

Nested values are supported. Separate nesting levels with a period. For example, configure the widget as follows to aggregate entities by status:

| Table name | Field name      |
| ---------- | --------------- |
| `entities` | `health.status` |

## Aggregation types

### Date

Chart data is sorted and grouped by the selected time intervals.

You can configure the following aggregation parameters:

- **Step unit** — For example, seconds, minutes, hours, or days.
- **Units per step** — For example, 5 minutes, 2 hours, or 1 day.

These parameters control the time-based granularity of the displayed data.

### Value

Chart data is sorted by value. For each unique value in the source data:

- The number of occurrences is counted.
- The chart displays a value-count pair.

This aggregation shows the distribution and frequency of values.

### Interval ranges

The **Interval ranges** aggregation divides values into configured numeric ranges. Use it to build histograms and analyze data distributions.

Configure intervals in one of two modes:

1. **Automatic division by interval count**.

   Specify the number of intervals into which the available data is divided. Intervals are calculated automatically and distributed evenly from the minimum to the maximum value.

1. **Manual interval boundaries**.

   Specify an array of numeric interval boundaries. For example: `0, 10, 20, 50`.

   The numbers are sorted in ascending order and form these intervals: `[0, 10)`, `[10, 20)`, `[20, 50]`.

Specify at least one of these aggregation parameters:

- `Count` — Number of intervals.
- `Boundaries` — Interval boundaries.

Examples:

- `Count = 5` — Creates five equal intervals based on the data.
- `Boundaries = 100, 0, 50` — Sorts the boundaries to `[0, 50, 100]` and creates the intervals `[0, 50)` and `[50, 100]`.
