---
title: ClickHouse. Table
weight: 30
---

The widget displays the result of a read-only SQL query to ClickHouse as a sortable, paginated table.
Use the `{{from}}` and `{{to}}` placeholders in the query.

## Configuration

| Name          | Required | Description                                                                                       | Default value |
| ------------- | -------- | ------------------------------------------------------------------------------------------------- | ------------- |
| Query         | Yes      | Read-only SQL query. Use `{{from}}` and `{{to}}` to specify the time range                        | —             |
| Database      | No       | ClickHouse database name passed in the `X-ClickHouse-Database` header                             | —             |
| Default range | No       | Range used when opening or refreshing the widget if no range is specified in the query parameters | Last hour     |
| Page size     | Yes      | Number of rows loaded by each query                                                               | 50            |

## Authorization

Authorization is described in [External services](../../external-services/#clickhouse).
