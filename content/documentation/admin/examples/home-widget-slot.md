---
title: Widget on a microservice tab
weight: 33
description: Showing a GitLab widget on the "Pipelines" and "MRs" tabs of a microservice card on the DDP Home page — widget configuration with the microservice context and filling in widget slots in mapping rules.
---

This guide describes showing the "GitLab. Pipelines" widget on the "Pipelines" tab of a microservice card.
The widget shows the pipelines of the repository of the microservice whose card is open.

A single widget serves all microservices because the widget gets the GitLab project ID from the card context, not from its own configuration.

## Before you begin

To follow this guide, prepare the following:

- Permissions to create widgets and the `update:home-mapping-sets` permission.
- A GitLab external service.
- A property with the numeric GitLab project ID on microservice entities, mapped to the `microservice_gitlab_project_id` role.
  In the canonical catalog model, this is the `gitlab_repository_id` property, which is filled in by the process that creates a microservice from a template or by the "GitLab: repository activity" datasource.

The "GitLab: widgets" dataset creates ready-made widgets for both tabs.
This guide shows how to configure the widget yourself, for example, for a different version control system or with different properties.

## Creating a widget

Create a widget with the project ID from the context:

1. Create a widget of the ["GitLab. Pipelines"](../widgets/gitlab/pipelines/) type with the GitLab external service.
1. In the "Project ID" field, specify `{{ .context.repository.id }}`.
1. Save the widget.

On a microservice card tab, the Home page passes the following values to the widget:

| Path | Value |
| --- | --- |
| `.context.microservice.uuid` | UUID of the microservice entity |
| `.context.microservice.slug` | Microservice identifier |
| `.context.microservice.name` | Microservice name |
| `.context.repository.id` | Value of the `microservice_gitlab_project_id` role |
| `.context.repository.url` | Value of the `microservice_repository` role |

The context is passed only on microservice card tabs.
For dashboards, create a separate widget with a specific project ID.

## Filling in the widget slot

Add the widget to the "Pipelines" tab slot:

1. Go to "Administration" → "Directories" → "Mapping rules" and open the default set.
1. On the "Catalog mapping" tab, expand the "Microservices" area and find the "Widgets" block.
1. In the "'Pipelines' tab widget" field, select the created widget.
1. Click "Save".

The "MRs" tab is filled in with the "'MRs' tab widget" field, for example, with the ["GitLab. Merge requests"](../widgets/gitlab/merge-requests/) widget with the same "Project ID" field value.

## Verifying the result

Check the widget on the Home page:

1. Open the Home page → "Microservices" and select a microservice in the table.
1. Open the "Pipelines" tab on the microservice card.

The tab shows the pipelines of the selected microservice's repository.
When you select another microservice, the widget shows the pipelines of its repository.

If the tab shows "This section is not available yet", the widget slot is not filled in.
If the widget shows no data, check that the microservice has the property mapped to the `microservice_gitlab_project_id` role filled in.
