---
title: Platform agent
weight: 20
description: Preparing the ddp-agent platform agent for use, user access, MCP collections, skills, and troubleshooting.
---

The platform agent with the `ddp-agent` ID is deployed together with Deckhouse Development Portal (DDP).
It is available in the AI assistant, the "AI chat" widget, and process steps.

The agent uses a language model through the AI provider of the running user.
In the catalog, it is marked as "DDP model"; the "Allow the DDP model proxy" switch in its connection is enabled and cannot be changed.
To work with project files, use a workspace.

## Quick start

To let users work with the agent and catalog data, follow these steps:

1. Grant access to the agent with the global `use:ai-agents` permission or the object-level `use:ai-agent` permission.
   For the "AI" section, add `view:ai-page`.
1. Prepare an [AI provider](../../user/ai-assistant/#connecting-an-ai-provider) or a [template with a shared token](provider-templates/).
1. Create an enabled MCP collection with the `ddp_get_resources` and `ddp_get_resource_entities` tools.
1. Grant the user permission to use this collection and read permissions on the required catalog objects.
1. In the agent catalog, open the platform agent for editing.
   On the "Tools and skills" tab, select the collection in "MCP collections" and save.
1. In the AI assistant, select the agent and the provider, then ask it to show the available catalog resources.

An empty set of collections means the agent works without MCP tools.
The portal does not automatically restore the previous set of read tools.

## Changing tools and skills

To extend the agent's capabilities, edit the collection or attach another one.
For example, for processes, add `get_processes`, `run_process`, and `get_process_instance`.
The user also needs access to these tools through collections and permissions for operations on entities.

On the "Tools and skills" tab, you can assign [skills](skills/).
To read them, `get_skill` must be available in the run.

When you add an entire portal group, new tools of that group are automatically included in the collection.
If the collection contents must stay fixed, select individual tools.

## Deployment and settings

The `development-platform` module creates the `agent` Deployment in `d8-development-platform`.
The agent is available inside the cluster at `http://agent:8080`; it does not need a separate Ingress.

The module sets the API key and the card URL.
The portal creates the connection automatically and checks it when the backend starts and once a minute.
The name, description, enabled state, collections, and skills are set in the connection.
The `ddp-agent` ID, the card URL, and the key are managed by the installation.

Disabling the agent removes it from selection in the chat, widgets, and processes.
The connection check does not re-enable it.
You cannot delete the platform agent connection.

## Troubleshooting

Check the agent status using the portal endpoint:

```shell
curl -s https://ddp.example.com/healthz
```

The `agents` field shows the status:

| Value | Meaning for the user |
| --- | --- |
| `ok` | The card can be read, the agent is ready |
| `unavailable` | The connection exists, but the card is unavailable |
| `not_configured` | The connection has not been created |

If the agent is unavailable, check the pod and its logs:

```shell
d8 k -n d8-development-platform get pods -l app=agent
d8 k -n d8-development-platform logs deploy/agent
```

If access is denied, check `use:ai-agents` or `use:ai-agent`.
If the agent responds without catalog data, check the agent's collections, the user's access to the tools, and the user's permissions on entities.
Check model errors in the AI provider and the [run history](runs/).
