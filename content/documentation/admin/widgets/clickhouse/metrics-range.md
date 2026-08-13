---
title: ClickHouse. Metrics (range)
description: Time-series chart based on a ClickHouse SQL query.
weight: 10
---

The widget plots a line chart based on a read-only SQL query to ClickHouse.
Use the `{{from}}` and `{{to}}` placeholders in the query to specify the time range boundaries.

Example of a valid widget query:

```sql
SELECT
  toStartOfMinute(timestamp) AS time,
  avg(value) AS value,
  service AS series
FROM metrics
WHERE timestamp >= {{from}} AND timestamp < {{to}}
GROUP BY time, series
ORDER BY time
```

## Configuration

| Name          | Required | Description                                                                                     | Default value |
| ------------- | -------- | ----------------------------------------------------------------------------------------------- | ------------- |
| Query         | Yes      | Read-only SQL query. Use `{{from}}` and `{{to}}` to specify the time range                       | —             |
| Database      | No       | ClickHouse database name passed in the `X-ClickHouse-Database` header                            | —             |
| Default range | No       | Range used when opening or refreshing the widget if no range is specified in the query parameters | Last hour     |
| Time column   | Yes      | Column containing timestamps for the chart's X-axis                                             | —             |
| Value column  | Yes      | Column containing numeric values for the chart's Y-axis                                         | —             |
| Series column | No       | Column used as the series name in the chart legend                                              | —             |
| Threshold     | No       | Threshold displayed as a horizontal line on the chart                                           | —             |
| Minimum value | No       | Starting point for the chart's vertical axis                                                    | —             |
| Maximum value | No       | End point for the chart's vertical axis                                                         | —             |


## Authorization

Authorization is described in [External services](../../external-services/#clickhouse).
