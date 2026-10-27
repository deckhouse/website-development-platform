---
title: Overview
description: Contents of the Deckhouse Development Portal administrator guide.
weight: 10
---

The administrator guide of Deckhouse Development Portal (DDP) describes how to install the portal and configure its mechanisms: datasources, actions, processes, widgets, health checks, and others.
The guide is part of the operational documentation for the product.

## Guide contents

The guide consists of the following sections:

| Section | Contents |
| --- | --- |
| [Installation](install/) | Installing the portal with internal, external, or managed PostgreSQL, Redis, and ClickHouse instances |
| [Compute resource requirements](resource-requirements/) | CPU, memory, and disk sizing for different load profiles |
| [Architecture](architecture/overview/) | Portal components, network interactions, and workers |
| [Security](security/overview/) | Role model, credentials, encryption key, audit logs, and HTTP security headers |
| [Home](home/overview/) | The portal Home page, mapping rules, and the initial setup wizard |
| [Microservice templates](templates/overview/) | Template registry, template packs, and updating microservices from a template |
| [Actions](actions/overview/) | Configuring actions and the reference of built-in actions |
| [Processes](processes/overview/) | Creating processes, element types, the process store, and execution management |
| [AI agents](ai-agents/overview/) | Connecting AI agents, skills, and run history |
| [Datasources](datasources/overview/) | Syncing entities from external systems and data processing rules |
| [Widgets](widgets/overview/) | Widgets for dashboards and entity pages |
| [Health checks](healthchecks/overview/) | Rules that determine the status of entities |
| [Automations](automations/) | Running actions, processes, and syncs automatically when entities change |
| [Webhooks](webhooks/) | Receiving data from external systems using the push model |
| [External services](external-services/) | Authentication in external infrastructure services |
| [Trusted certificates](trusted-certificates/) | Certificates for HTTPS connections to external services |
| [Datasets](datasets/) | Ready-made configurations for integration with external systems |
| [Workflows](workflows/) | Chaining actions together |
| [Workspaces](workspaces/overview/) | Preparing workspaces, workspace templates, and template sources |
| [Configuration examples](examples/overview/) | Step-by-step portal configuration scenarios |

## New scenarios

To configure new scenarios, use the following guides:

- [Preparing workspaces](workspaces/overview/): Connections, network, storage, and limits.
- [Creating a template](workspaces/templates/): OpenTofu project, parameters, publishing, and activation.
- [Files and scripts](workspaces/files/) and [images, Ingress, and storage](workspaces/network-storage/): Examples of modifying the starter template.
- [AI agents](ai-agents/overview/) and [skills](ai-agents/skills/): Agent tools and instructions.
