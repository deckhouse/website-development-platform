---
title: Overview
description: Widget purpose, scope, credentials, and configuration principles
weight: 10
---

Widgets are cards that visualize data stored in Deckhouse Development Portal (DDP) and information from infrastructure services.
Unlike datasources, widgets retrieve information from infrastructure services when they are displayed in the interface.

Widgets can be added to dashboards.
Dashboards can be linked to static pages (**Catalog**, **Self-service**, **AI**, **Administration**, **RBPO**) and to entity cards.

A widget is also displayed on the **Home** page in a widget slot of a microservice card.

## Configuration

A widget configuration includes common parameters and fields specific to the widget type.

Widget configurations support [Go template](https://pkg.go.dev/text/template) syntax for templating during widget processing.
For example:

* `{{ .entity.name }}`: Substitutes the value of the entity's `name` property.
* `{{ .credentials.token }}`: Substitutes credentials named `token`.
* `{{ .context.repository.id }}`: Substitutes a value from the Home page context.
  The context is passed only to widgets in Home page slots.

You can set a scope for each widget.
A widget with the `Global` scope cannot retrieve entity properties using Go templates.
A widget with the `Resource` scope can retrieve entity properties using Go templates and can only be attached to entity pages.

In the widget configuration, you can specify the account whose credentials the widget uses to interact with infrastructure systems and select the credentials type.

If no account is specified, the widget uses the credentials of the current user.
