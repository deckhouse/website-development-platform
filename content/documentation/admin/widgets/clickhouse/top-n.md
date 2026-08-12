---
title: ClickHouse. Top N
description: Horizontal bar chart based on a ClickHouse SQL query.
weight: 40
---

The widget plots a horizontal bar chart based on a read-only SQL query to ClickHouse.
The query must return columns containing labels and numeric values.
The **Limit** parameter restricts the number of displayed rows.

Example of a valid widget query:

```sql
SELECT service AS label, count() AS value
FROM events
WHERE timestamp >= {{from}} AND timestamp < {{to}}
GROUP BY label
ORDER BY value DESC
```

## Configuration

| Name          | Required | Description                                                                                       | Default value |
| ------------- | -------- | ------------------------------------------------------------------------------------------------- | ------------- |
| Query         | Yes      | Read-only SQL query. Use `{{from}}` and `{{to}}` to specify the time range                        | —             |
| Database      | No       | ClickHouse database name passed in the `X-ClickHouse-Database` header                             | —             |
| Default range | No       | Range used when opening or refreshing the widget if no range is specified in the query parameters | Last hour     |
| Label column  | Yes      | Column containing the bar labels                                                                  | —             |
| Value column  | Yes      | Column containing numeric values that determine bar length                                        | —             |
| Limit         | Yes      | Maximum number of rows in the chart                                                               | 10            |

## Authorization

Authorization is described in [External services](../../external-services/#clickhouse).
