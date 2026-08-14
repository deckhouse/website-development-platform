---
title: Prometheus. Metrics (range)
description: Time-series chart based on a PromQL range query to Prometheus.
weight: 10
---

The widget plots a chart from the result of a Prometheus `query_range` query.
The PromQL query must return a `Matrix` type containing one or more time series.

Example query with placeholders:

```promql
sum by (status) (rate(http_requests_total[{{rateInterval}}]))
```

## Query placeholders

The widget substitutes values calculated from the active time range:

| Placeholder        | Description                                                        |
| ------------------ | ------------------------------------------------------------------ |
| `{{range}}`        | Duration of the selected time range                                |
| `{{rateInterval}}` | Range window for the `rate()` and `increase()` functions           |
| `{{interval}}`     | Interval between chart points, calculated automatically            |

The query step is selected based on the time range duration.
The chart contains no more than 1100 points, and the minimum step is 30 seconds.
The **Resolution step** configuration field is no longer used.

## Configuration

| Name               | Required | Description                                                                                                     | Default value |
| ------------------ | -------- | --------------------------------------------------------------------------------------------------------------- | ------------- |
| URL                | Yes      | Prometheus URL                                                                                                  | —             |
| Query              | Yes      | PromQL query for the range chart                                                                                | —             |
| Label              | Yes      | Prometheus label name used as the series name in the chart legend                                               | —             |
| Default range      | No       | Range used when loading or refreshing the widget unless the user selects another range in the widget panel      | Last hour     |
| Threshold          | No       | Horizontal dashed line on the chart                                                                             | —             |
| Minimum value      | No       | Lower Y-axis boundary. Leave empty to scale automatically                                                       | —             |
| Maximum value      | No       | Upper Y-axis boundary. Leave empty to scale automatically                                                       | —             |
| `InsecureSkipVerify` | No     | Disables verification of the Prometheus TLS/SSL certificate                                                     | `false`       |

## Authorization

Authorization is described in [External services](../../external-services/#prometheus).
