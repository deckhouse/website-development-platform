---
title: Prometheus. Metrics (single value)
weight: 20
---

The widget displays a single number from a PromQL query to Prometheus.
The query must return a **Scalar** or a **Vector** containing one value.

Example query without placeholders:

```promql
sum(machine_cpu_cores)
```

Example with a placeholder for the time range duration selected in the widget panel:

```promql
sum(increase(http_requests_total[{{range}}]))
```

## Query placeholders

Use placeholders when the expression requires the duration of the time range selected in the widget panel.
The query runs at the end of this time range.

| Placeholder        | Description                                               |
| ------------------ | --------------------------------------------------------- |
| `{{range}}`        | Duration of the selected time range                       |
| `{{rateInterval}}` | Range window for the `rate()` and `increase()` functions  |
| `{{interval}}`     | Query step calculated from the time range                 |

## Configuration

| Name                         | Required | Description                                                                                                             | Default value |
| ---------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------- | ------------- |
| URL                          | Yes      | Prometheus URL                                                                                                          | —             |
| Query                        | Yes      | PromQL query that returns one value                                                                                     | —             |
| Default range                | No       | Range used when loading or refreshing the widget unless the user selects another range in the widget panel             | Last hour     |
| Decimal places               | No       | Precision used to display the returned value                                                                           | —             |
| Unit                         | No       | Suffix displayed with the returned value                                                                                | —             |
| Show threshold               | No       | Displays the threshold as `<metric value> / <threshold>`                                                                | false         |
| Threshold                    | No       | Threshold value                                                                                                         | —             |
| Lower value is better        | No       | Considers the metric healthy when its value is below the configured threshold                                          | false         |
| Warning threshold (%)        | No       | Boundary between red and orange. A metric value above this percentage of the threshold is displayed in orange          | 60            |
| Success threshold (%)        | No       | Boundary between orange and green. A metric value above this percentage of the threshold is displayed in green         | 90            |
| InsecureSkipVerify           | No       | Disables verification of the Prometheus TLS/SSL certificate                                                            | false         |

## Authorization

Authorization is described in [External services](../../external-services/#prometheus).
