---
title: Overview
weight: 10
description: DDP AI agent types, access model, MCP collections, skills, and running agents in the chat, widgets, and processes.
---

{{< alert level="warning" >}}
AI agents are a Preview feature.
Agent types, available settings, and the API may change in future releases without backward compatibility.
{{< /alert >}}

An AI agent receives a task in natural language, calls a language model, and invokes tools to prepare an answer or perform the work.
Deckhouse Development Portal (DDP) connects agents over the A2A (Agent2Agent) protocol version 1.0.

Agents from the catalog are used in the [AI assistant](../../user/ai-assistant/), the ["AI chat" widget](../widgets/ai/chat/), and the ["AI agent" process element](../processes/overview/#ai-agent).
A separate workspace agent is also available for working with project files.

## Agent types

An agent connection is defined by its Agent Card, which the agent publishes as JSON, usually at `/.well-known/agent-card.json`:

| Type | Connection |
| --- | --- |
| [Platform agent](platform-agent/) | Deployed with the portal and connected automatically, ID `ddp-agent` |
| [External agent](external-agents/) | Connected by an administrator using the card URL and an API key |
| [Agent from a source](agent-sources/) | Created by a source from a catalog entity |

The portal sends a message to the agent over A2A and receives a response. The agent itself runs outside the portal backend.

## Agent language model

An agent uses its own model access or the DDP model proxy.
When working through the proxy, the portal uses the AI provider of the user who started the task and consumes that user's quota.

In the catalog, the model source is shown as "DDP model" or "Agent model".

The platform agent always uses the proxy.
For an external agent, enable "Allow the DDP model proxy" in the connection settings if the portal must provide the model.

The proxy works with an API compatible with OpenAI Chat Completions.
`/chat/completions` is appended to the provider URL unless the URL already ends with it.
A model and a token are required; the provider fields "Request body template" and "Response field" are not used for agent requests.

The proxy does not support `stream: true` and substitutes the AI provider's model for the model specified in the agent request.

## Agent tools

On the "Tools and skills" tab, set "MCP collections".
The agent receives the tools of the enabled collections that the running user is allowed to use through their own collections.
Permissions on portal objects are checked separately on every call.

Without collections, the agent works without MCP tools.
This rule is the same for the platform agent, external agents, the chat, widgets, and processes.
A collection can include portal tools, tools of connected MCP servers, and custom HTTP tools.

For example, to let the agent read the catalog, add `ddp_get_resources` and `ddp_get_resource_entities` to a collection, attach the collection to the agent, and grant the user permission to use it.
To run processes, add `get_processes`, `run_process`, and `get_process_instance`.

For step-by-step setup, see the [MCP collections guide](../../user/mcp-management/#mcp-collections).

## Agent skills

On the same tab, assign [skills](skills/) with a specific version or the "Latest" binding.
At run time, the agent receives their names and descriptions and reads the full text through `get_skill` when needed.
This tool must be available through both the agent's and the user's collections.

For instructions and materials stored in GitLab, configure a [skill source](skill-sources/).

## Access to agents

Usage is controlled by the global `use:ai-agents` permission or the object-level `use:ai-agent` permission for a specific connection.
The permission applies to the chat, widgets, and process steps.
It does not grant access to catalog data or provide MCP tools.

Viewing and editing are controlled by separate permissions:

| Object | Global permissions | Object-level permissions |
| --- | --- | --- |
| Agent | `read:ai-agents`, `edit:ai-agents` | `read:ai-agent`, `edit:ai-agent`, `use:ai-agent` |
| Agent source | `read:ai-agents`, `edit:ai-agents` | `read:ai-agent-source`, `edit:ai-agent-source` |
| AI provider template | `read:ai-provider-templates`, `edit:ai-provider-templates`, `use:ai-provider-templates` | `read:ai-provider-template`, `edit:ai-provider-template`, `use:ai-provider-template` |

Ownership and membership in the owner team grant permissions when `owner-is-admin` is enabled.
You can grant access to other users and teams using "Edit role bindings".
A super administrator has access to all connections.

Viewing your own run history requires `read:ai-agent-runs`, and stopping runs requires `cancel:ai-agent-runs`.
The "AI" section requires `view:ai-page`; this permission alone does not grant access to the section's objects.

## Run token and limits

For permitted calls to MCP and the model proxy, the portal passes a short-lived run token.
The token is revoked after the run completes or is stopped.

The following limits apply to A2A agent runs:

| Limit | Value |
| --- | --- |
| Maximum run duration | 70 minutes |
| Maximum token lifetime | 75 minutes |
| MCP calls | 120 per minute per run |
| Model proxy requests | 60 per minute per run |
| Proxy request size | 1 MiB |
| Model response size through the proxy | 4 MiB |
| Model request timeout | 90 seconds |

Check errors and results in the [run history](runs/), and aggregated metrics in the AI statistics.
