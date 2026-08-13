---
title: ClickHouse. Metrics (single value)
description: Single-value metric based on a ClickHouse SQL query.
weight: 20
---

The widget displays a single number based on a read-only SQL query to ClickHouse.
You can specify a unit and configure a threshold for the value.
The query must return one row; the widget displays the value from the first column.

Example of a valid widget query:

```sql
SELECT count() AS value
FROM events
WHERE timestamp >= {{from}} AND timestamp < {{to}}
```

## Configuration

| Name                         | Required | Description                                                                                                             | Default value |
| ---------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------- | ------------- |
| Query                        | Yes      | Read-only SQL query. Use `{{from}}` and `{{to}}` to specify the time range                                              | —             |
| Database                     | No       | ClickHouse database name passed in the `X-ClickHouse-Database` header                                                   | —             |
| Default range                | No       | Range used when opening or refreshing the widget if no range is specified in the query parameters                      | Last hour     |
| Decimal places               | No       | Precision used to display the returned value                                                                           | —             |
| Unit                         | No       | Suffix displayed with the returned value                                                                                | —             |
| Show threshold               | No       | Displays `<METRIC_VALUE> / <THRESHOLD>`, where `<METRIC_VALUE>` is the current metric value and `<THRESHOLD>` is the configured threshold | `false`       |
| Threshold                    | No       | Threshold value                                                                                                         | —             |
| Lower value is better        | No       | Considers the metric healthy when its value is below the configured threshold                                          | `false`       |
| Warning threshold (%)        | No       | Boundary between red and orange. A metric value above this percentage of the threshold is displayed in orange          | 60            |
| Success threshold (%)        | No       | Boundary between orange and green. A metric value above this percentage of the threshold is displayed in green         | 90            |

## Authorization

Authorization is described in [External services](../../external-services/#clickhouse).
