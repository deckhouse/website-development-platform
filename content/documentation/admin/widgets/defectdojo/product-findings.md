---
title: DefectDojo. Product vulnerability details
description: Detailed product vulnerabilities retrieved from DefectDojo
weight: 20
---

The widget displays a table of product vulnerabilities based on DefectDojo data.
For each vulnerability, the table includes its severity, description, and detection date.

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
