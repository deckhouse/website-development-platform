---
title: DefectDojo. Product vulnerabilities summary
description: Product vulnerability totals grouped by severity
weight: 30
---

The widget displays a chart with the total number of product vulnerabilities from DefectDojo, grouped by severity.

## Configuration

When adding the widget, specify the following parameters:

| Name         | Required | Description                                                  | Default value |
| ------------ | -------- | ------------------------------------------------------------ | ------------- |
| URL          | Yes      | DefectDojo URL without the API path (`/api/v2`)              | —             |
| Product name | Yes      | Product name in DefectDojo                                   | —             |

## Additional widget features

When viewing the widget, configure the following parameters.

If **Active vulnerabilities** is enabled, the widget loads product vulnerabilities with the `Active` flag set to `true`.
If it is disabled, the widget loads vulnerabilities with the `Active` flag set to `false`.
The parameter is enabled by default.

If **Duplicate vulnerabilities** is enabled, the widget loads product vulnerabilities with the `Duplicate` flag set to `true`.
If it is disabled, the widget loads vulnerabilities with the `Duplicate` flag set to `false`.
The parameter is disabled by default.

## Authorization

Authentication is described in [External services](../../external-services/#defectdojo).
