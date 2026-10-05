---
title: Interface
weight: 20
---

The Deckhouse Development Portal (DDP, portal) interface consists of the following sections:

- "Home": the developer workspace with teams, systems, microservices, environments, deployment, and resource ordering. The tab is displayed if the administrator has enabled the home page.
- "Catalog": a service catalog for viewing resources, entities, and relationships, and for running actions and scenarios.
- "Self-service": a section for configuring data sources, actions, webhooks, automations, scenarios, dashboards, and widgets. Access to this section can be restricted by the RBAC model.
- "AI": a section for configuring MCP servers, custom tools, and tool collections. For details, refer to [MCP management](mcp-management/). Access to this section can be restricted by the RBAC model.
- "Administration": a section for managing teams, users, access policies, and credentials. Access to this section can be restricted by the RBAC model.

## Top bar

The right side of the portal top bar contains the following buttons:

- "AI Assistant": opens the [AI assistant](ai-assistant/#using-the-ai-assistant) window.
- "My Activities": opens a panel with the runs of your actions and processes. While your processes are running, the button icon is replaced with an indicator, and the "Processes" tab shows the number of running processes.

## Global search

At the top of the "Catalog" sidebar, a "Search" field lets you quickly find entities across the portal.

### Limitations

- Maximum query length: 255 characters.
- Search works only for entities. Resources, teams, and other portal objects are not included in search results.
