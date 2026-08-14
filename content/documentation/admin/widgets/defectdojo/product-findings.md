---
title: DefectDojo. Product vulnerability details
description: Detailed product vulnerabilities retrieved from DefectDojo
weight: 20
---

The widget displays a table of product vulnerabilities based on DefectDojo data. For each vulnerability, the table includes its severity, description, and detection date.

## Configuration

| Name         | Required | Description                                                  | Default value |
| ------------ | -------- | ------------------------------------------------------------ | ------------- |
| URL          | Yes      | DefectDojo URL without the API path (`/api/v2`)              | —             |
| Product name | Yes      | Product name in DefectDojo                                   | —             |

## Additional widget features

When viewing the widget, configure the following parameters:

- **Active vulnerabilities** — when enabled, loads product vulnerabilities with the `Active` flag set to `true`. When disabled, loads vulnerabilities with the `Active` flag set to `false`. Enabled by default.
- **Duplicate vulnerabilities** — when enabled, loads product vulnerabilities with the `Duplicate` flag set to `true`. When disabled, loads vulnerabilities with the `Duplicate` flag set to `false`. Disabled by default.

## Authentication

Authentication is described in [External services](../../external-services/#defectdojo).
