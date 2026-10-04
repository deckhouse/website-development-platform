---
title: Overview
description: Widget purpose, scope, credentials, and configuration principles
weight: 10
---

Widgets are cards that visualize data stored in Deckhouse Development Portal (DDP) and information from infrastructure services. Unlike data sources, widgets retrieve information from infrastructure services when they are displayed in the interface.

Widgets can be added to dashboards. Dashboards can be linked to:

- static pages: **Catalog**, **Self-service**, **AI**, **Administration**, **RBPO**;
- entity cards.

A widget is also displayed on the **Home** page in a widget slot of a microservice card.

## Configuration

A widget configuration includes common parameters and fields specific to the widget type.

Widget configurations support [Go template](https://pkg.go.dev/text/template) syntax for templating during widget processing. For example:

* `{{ .entity.name }}` — substitutes the value of the entity's `name` parameter.
* `{{ .credentials.token }}` — substitutes credentials named `token`.
* `{{ .context.repository.id }}` — substitutes a value from the home page context. The context is passed only to widgets in home page slots.

You can set a scope for each widget:

* `Global` — the widget cannot retrieve entity parameters using Go templates.
* `Resource` — the widget can retrieve entity parameters using Go templates. Widgets with the `Resource` scope can only be attached to entity pages.

In the widget configuration, you can specify the account whose credentials the widget uses to interact with infrastructure systems and select the credentials type.

If no account is specified, the widget uses the credentials of the current user.
