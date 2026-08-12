---
title: Overview
weight: 10
---

Widgets are cards that visualize data stored in the platform and information from infrastructure services. Unlike data sources, widgets retrieve information from infrastructure services when they are displayed in the interface.

Widgets can be added to dashboards. Dashboards can be linked to:

- static pages, such as the catalog, self-service, home, and administration pages;
- entity cards.

## Configuration

A widget configuration includes common parameters and fields specific to the widget type.

Widget configurations support [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) syntax for templating during widget processing. For example:

* `{{ .entity.name }}` — substitutes the value of the entity's `name` parameter.
* `{{ .credentials.token }}` — substitutes credentials named `token`.

You can set a scope for each widget:

* `Global` — the widget cannot retrieve entity parameters using [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax).
* `Resource` — the widget can retrieve entity parameters using [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax). Widgets with the `Resource` scope can only be attached to entity pages.

In the widget configuration, you can specify the account whose credentials the widget uses to interact with infrastructure systems and select the credentials type.

If no account is specified, the widget uses the credentials of the current user.
