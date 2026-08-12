---
title: Jira
description: Display and filter Jira issues with configurable JQL queries.
weight: 110
---

The widget displays Jira issues based on a JQL query.

## Authorization

Authorization is configured in [External services](../../external-services/#jira).

## Configuration

| Name | Required | Description                                                                 | Default |
| ---- | -------- | --------------------------------------------------------------------------- | ------- |
| URL  | Yes      | Jira URL used to retrieve data                                              | —       |
| JQL  | Yes      | JQL query used to filter issues. Example: `project = PROJ AND status = Open` | —      |

## Query parameters

| Name            | Required | Description                                                              | Default            |
| --------------- | -------- | ------------------------------------------------------------------------ | ------------------ |
| JQL             | No       | JQL query used to filter issues. If omitted, the configured JQL is used | From configuration |
| Maximum results | No       | Maximum number of issues to display, from 1 to 1000                     | 50                 |

## Additional widget features

* **View description** — Opens a dialog with the full issue description.
* **Open in Jira** — Opens the issue in a new tab when you select its key.
* **Dynamic filtering** — Lets you change the JQL query and maximum number of results directly in the widget without changing its configuration.
