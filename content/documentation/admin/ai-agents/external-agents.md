---
title: External agents
weight: 30
description: Connecting an external AI agent over the A2A protocol — connection parameters, agent requirements, and connecting kagent agents.
---

{{< alert level="warning" >}}
AI agents are a Preview feature.
Agent types, available settings, and the API may change in future releases without backward compatibility.
{{< /alert >}}

An external agent is an AI agent that runs outside the portal and accepts tasks over the A2A protocol version 1.0.
The portal connects to the agent using its card URL and an API key, after which the agent can be selected in the AI assistant, the "AI chat" widget, and the "AI agent" process element.

If there are many agents and they are already described in the catalog, connect them through an [agent source](agent-sources/): the source creates and updates the connections itself.

## Quick start

To connect an external agent, follow these steps:

1. Go to "AI" → "Agents" → "Catalog" and click "Connect agent".
   The button is available with the `edit:ai-agents` permission.
1. On the "Main information" tab, fill in "Name" and "Identifier".
1. On the "Connection" tab, turn on the "Enable" switch.
1. Specify the "Agent Card URL", that is, the agent card address, for example, `https://agents.example.com/.well-known/agent-card.json`.
1. Specify the "API key" that the agent expects.
1. Click "Save".

The agent appears in the catalog with the "External" mark.
To let a user run it, grant the user `use:ai-agents` or `use:ai-agent` on this connection.
Ownership grants permissions when `owner-is-admin` is enabled.

## Connection parameters

The connection form contains the "Main information", "Connection", and "Tools and skills" tabs.

### Basic information

The "Main information" tab contains the following fields:

| Name | Required | Description |
| --- | --- | --- |
| "Name" | Yes | Agent name in the catalog and in agent selection lists |
| "Identifier" | Yes | From 3 to 64 characters: lowercase Latin letters, digits, and hyphens. The `ddp-agent` ID is reserved for the platform agent |
| "Description" | No | Agent description |

The ID is used to specify the agent in the "AI agent" process element and in the "AI chat" widget settings.

### Connection

The fields of the "Connection" tab are shown when the "Enable" switch is on:

| Name | Required | Description |
| --- | --- | --- |
| "Enable" | — | A disabled agent is not shown in the AI assistant, widgets, and process steps |
| "Agent Card URL" | Yes | Agent card address. The portal reads the name, description, and the addresses at which the agent accepts calls from the card |
| "Call the address from the agent card" | — | If enabled, the portal calls the address declared in the card. If disabled, the portal calls the base URL of the card, that is, the address from the "Agent Card URL" field without `/.well-known/agent-card.json` or `/.well-known/agent.json` |
| "Allow the DDP model proxy" | — | The agent accesses the language model through the portal and uses the AI provider of the user who started the task. For details, see ["Agent language model"](overview/#agent-language-model) |
| "API key" | Yes | Key that the portal passes to the agent in the `X-A2A-Api-Key` and `Authorization: Bearer` headers on every call and when reading the card |

On the "Tools and skills" tab, select "MCP collections" and, if needed, [skills](skills/).
The agent receives the tools of these collections that the running user is allowed to use.
Without collections, the agent works without MCP tools.
Access to tools is defined by [collections and roles](../../user/mcp-management/#integration-with-the-ai-assistant-and-mcp-server).

The API key is stored encrypted in the database and is not displayed after saving.
When editing the connection, leave the "API key" field empty to keep the current key.
During [encryption key rotation](../security/encryption-key-rotation/), agent API keys are re-encrypted.

## Agent requirements

The portal works with an agent that meets the following requirements:

- It supports the A2A protocol version 1.0 with the HTTP+JSON or JSON-RPC binding.
- It publishes an agent card reachable from the portal cluster.
- It accepts the API key from the `X-A2A-Api-Key` or `Authorization: Bearer` header.

The portal sends an A2A message to the agent and waits for the task to complete, polling its state every 2 seconds.
The agent response is the text of its final message.

### Message metadata

In the metadata of each A2A message, the portal passes the access that the agent is allowed:

| Key | When passed | Value |
| --- | --- | --- |
| `ddp.mcp.endpoint` | The collections contain permitted tools | Portal MCP endpoint address, for example, `https://<DOMAIN>/api/v2/agent-mcp` |
| `ddp.mcp.runToken` | The collections contain permitted tools | Run token |
| `ddp.mcp.toolAllowlist` | The collections contain permitted tools | Tools permitted in this run |
| `ddp.llm.baseUrl` | The "Allow the DDP model proxy" switch is on | Model proxy address, for example, `https://<DOMAIN>/api/v2/agent-llm/v1` |
| `ddp.llm.apiKey` | The "Allow the DDP model proxy" switch is on | Run token |
| `ddp.llm.model` | The "Allow the DDP model proxy" switch is on | Model of the user's AI provider |
| `ddp.provider` | The "Allow the DDP model proxy" switch is on | Name of the user's AI provider |

The addresses in the metadata are built from the public portal address, so the agent must have network access to the portal.

### Calling portal tools

To call portal tools, the agent sends requests to the MCP endpoint from the metadata with the run token in the `X-Agent-Run-Token` header.
The `POST <ddp.mcp.endpoint>/tools/list` request returns the list of available tools with the JSON schema of their arguments.
The `POST <ddp.mcp.endpoint>/tools/call` request calls a tool; the request body is `{"tool": "<NAME>", "arguments": {...}}`.

Example of a tool call:

```shell
curl -s -X POST "https://<DOMAIN>/api/v2/agent-mcp/tools/call" -H "X-Agent-Run-Token: <RUN_TOKEN>" -H "Content-Type: application/json" -d '{"tool": "ddp_get_resources", "arguments": {}}'
```

Where:

- `<DOMAIN>`: Portal domain.
- `<RUN_TOKEN>`: Value of `ddp.mcp.runToken` from the message metadata.

A tool that is not in `ddp.mcp.toolAllowlist` returns a `403` error.
An invalid or revoked token returns a `401` error.

### Accessing the model through the portal

If the agent is allowed to use the Deckhouse Development Portal (DDP) model proxy, it sends requests to `<ddp.llm.baseUrl>/chat/completions` as to the OpenAI Chat Completions API and passes `ddp.llm.apiKey` as the API key.
Proxy limitations are described in ["Agent language model"](overview/#agent-language-model).

### Run result

The agent can report the result type in the metadata of the final message, task, or artifact.
The `ddp.deliverableType` field contains the result type: `message`, `merge_request`, `branch`, or `patch`.
The `ddp.deliverableRef` field contains a reference to the result, for example, the merge request URL.

The type and reference are shown in the run details in the [run history](runs/).
If the type is not passed, the result is considered a message.

## Connecting a kagent agent

Agents deployed in the cluster with kagent are connected through the kagent A2A endpoint.
The card of such an agent contains in-cluster service addresses, so the portal calls the base URL of the card.

To connect a kagent agent, follow these steps:

1. In the "Agent Card URL" field, specify the address of the agent's A2A endpoint in kagent, for example, `https://<KAGENT_HOST>/api/a2a/<NAMESPACE>/<AGENT_NAME>/`.
1. Turn off the "Call the address from the agent card" switch.
1. In the "API key" field, specify the key that the kagent A2A endpoint expects.
1. If the agent has no model access of its own, enable "Allow the DDP model proxy".

Where:

- `<KAGENT_HOST>`: Address of the kagent A2A endpoint.
- `<NAMESPACE>`: Agent namespace.
- `<AGENT_NAME>`: Agent name.

If there are several kagent agents, create an [agent source](agent-sources/) with the "Call the address from the agent card" switch turned off.

## Editing and deleting

You can edit and delete a connection with the global `edit:ai-agents` permission or the object-level `edit:ai-agent` permission.
When `owner-is-admin` is enabled, the owner and members of the owner team also get these permissions; a super administrator has access to all connections.
To edit or delete a connection, select "Edit" or "Delete" in the agent card menu in the catalog.

If you disable an agent manually, it is not re-enabled automatically, even if it is managed by an agent source.

## Troubleshooting

Agent call errors are shown in the AI assistant and in the run details:

| Message | Cause and solution |
| --- | --- |
| "Failed to get the external agent's Agent Card. Check the card URL and agent availability." | The card cannot be read at the address from the "Agent Card URL" field. Check the address and network access from the portal cluster to the agent |
| "The external agent's Agent Card is unusable (no supported interfaces or invalid address). Check the agent settings." | The card contains no addresses with a supported A2A binding. If the agent publishes addresses inside its own cluster, turn off "Call the address from the agent card" |
| "The external agent rejected the credentials. Check the API key of its link." | The agent did not accept the API key. Specify the key that the agent expects in the connection |
| "The external agent is unavailable (network error or timeout). Try again later." | The agent did not respond in time or is unreachable over the network |
| "The external agent returned no answer." | The agent task completed without a final message |
| "You do not have permission to run this AI agent" | Check the global `use:ai-agents` permission or the object-level `use:ai-agent` permission on the connection |
