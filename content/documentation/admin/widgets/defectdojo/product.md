---
title: DefectDojo. Product
description: Product vulnerabilities grouped by engagement and severity
weight: 10
---

The widget displays product vulnerabilities from DefectDojo, grouped by engagement and severity.

## Authentication

Authentication is described in [External services](../../external-services/#defectdojo).

## Configuration

| Name           | Required | Description                                                                                              | Default value                    |
| -------------- | -------- | -------------------------------------------------------------------------------------------------------- | -------------------------------- |
| URL            | Yes      | DefectDojo URL without the API path (`/api/v2`)                                                          | —                                         |
| Product name   | Yes      | Product name in DefectDojo                                                                              | —                                         |
| Severity levels | Yes     | Vulnerability severity levels loaded when the widget opens or when the selected engagement changes      | `Critical`, `High`, `Medium`, `Low`, `Info` |

## Request parameters

In the widget request settings, select the severity levels to load only vulnerabilities with those levels. By default, the widget uses the levels from its configuration. Changing the levels reloads data from DefectDojo.

## Additional widget features

### Filters

- **Engagement** — selects a product engagement. By default, the most recently created engagement, with the highest ID, is selected. Changing the engagement reloads the data.
- **Filters** — filters vulnerabilities by severity, tags, tests, and components. Tag, test, and component filters apply only to already loaded vulnerabilities and do not send new requests to DefectDojo.

### Tabs

- **Overview** — displays vulnerabilities by severity, tags (top 10), tests (top 10), and components (top 10). Charts use the filtered vulnerability set.
- **Details** — displays a table with details for each vulnerability.

{{< alert level="info" >}}
If the selected engagement and severity levels contain more than 1,000 vulnerabilities, the widget displays the first 1,000 records and a partial-load warning. Tag, test, and component filters apply only to the loaded set.
{{< /alert >}}
