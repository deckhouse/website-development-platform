---
title: OpenSearch
description: Search and inspect records from an OpenSearch index or index pattern.
weight: 40
---

The OpenSearch index widget displays data from a specific index or index pattern in the platform. By default, data is sorted from newest to oldest. Full-text search is available to filter the displayed data. Each record (table row) can be displayed as key-value pairs or as JSON. When an index pattern is specified, the widget provides a link to the Discover page in OpenSearch Dashboards.

## Configuration

| Name            | Required | Description                                                                                                                               | Default      |
| --------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| API URL         | Yes      | OpenSearch API URL used to retrieve data                                                                                                  | —            |
| Dashboards URL  | Yes      | OpenSearch Dashboards URL used to generate a link for viewing data directly in OpenSearch                                                 | —            |
| Index pattern   | Yes      | Index pattern from which the widget loads data. May contain `*`. Examples: `security-auditlog`, `security-auditlog-*`                     | —            |
| Timestamp field | No       | Name of the timestamp field. Its value is displayed in a separate column in the data table                                                | `@timestamp` |


## Authorization

Authorization is configured in [External services](../../external-services/#opensearch).
