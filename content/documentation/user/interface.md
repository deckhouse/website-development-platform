---
title: Interface
weight: 20
---

The Deckhouse Development Platform interface consists of the following sections:

- **Home**: the platform’s landing page. You can place one or more dashboards with widgets here.
- **Catalog**: a service catalog for viewing resources, entities, and relationships, and for running actions and scenarios.
- **Self-Service**: a section for configuring data sources, actions, webhooks, automations, scenarios, dashboards, and widgets. Access to this section can be restricted by the RBAC model.
- **Administration**: a section for managing teams, users, access policies, and credentials. Access to this section can be restricted by the RBAC model.

## Global search

The top navigation bar includes a **Search** button. It lets you quickly find entities across the platform.

### How to use search

1. Open search: click **Search** in the top navigation bar.
1. Enter a query: start typing in the search field.
1. Select a result: results appear in a dropdown list under the input field. For each matching entity, the platform shows:
   - name;
   - ID;
   - slug;
   - the resource the entity belongs to.

### Limitations

- Maximum query length: 255 characters.
- Search works only for entities. Resources, teams, and other platform objects are not included in search results.
