---
title: Agent sources
weight: 40
description: Connecting external AI agents automatically from catalog entities — resource bindings, filter rules, synchronization, and its results.
---

{{< alert level="warning" >}}
AI agents are a Preview feature.
Agent types, available settings, and the API may change in future releases without backward compatibility.
{{< /alert >}}

An agent source creates external agent connections from catalog entities.
Each entity of a bound resource becomes an external agent.
The agent ID is taken from the entity ID, and the card URL is taken from an entity property.

The source does not access external systems itself; it reads entities that already exist in the catalog.
Agent entities can be created manually, through the API, or by a [datasource](../datasources/overview/), for example, by synchronizing kagent agent objects from Kubernetes.

## How the source works

The portal synchronizes each enabled source when the backend starts and then once a minute.
You can start synchronization manually with the "Sync now" button.

For each resource binding, the portal performs the following steps:

1. Gets all entities of the resource.
1. Applies the [filter rules](#filter-rules) to the entities.
1. Skips entities whose [enablement property](#enablement-property) is turned off.
1. Creates or updates an agent connection for each remaining entity.

The portal takes the agent name and description from its card, and until the card has been read, from the entity name and description.
The portal re-reads the agent card if the agent is new, if the card URL has changed, or if more than 15 minutes have passed since the last read.

The owner and owner team of the entity become the owner and owner team of the agent.
If they are not set for the entity, the owner and owner team of the source are used.

Synchronization reflects the current state of the catalog as a whole:

- An agent whose entity has dropped out of the selection is disabled but not deleted.
- If the selection is empty, all agents of the source are disabled, and this is not considered an error.
- If the entity gets back into the selection, the agent is enabled.

## Quick start

To connect agents from the catalog, follow these steps:

1. Create a resource whose entities will describe agents.
   Add a `String` property for the agent card URL to the resource, for example, `agent_card_url`.
1. Create entities of the resource and fill in the card URL in them. The entity ID becomes the agent ID.
1. Go to "AI" → "Agents" → "Agent sources" and click "Create agent source".
   The button is available with the `edit:ai-agents` permission.
1. On the "Main information" tab, turn on "Enable" and fill in "Name" and "Identifier".
1. On the "Catalog resources" tab, click "Add resource", select the resource, and in the "Agent Card URL property" field, select the property with the card URL.
1. On the "Connection" tab, specify the "API key" shared by all agents of the source.
1. Click "Save", then select "Sync now" in the source card menu.

After synchronization, the portal shows the notification "Synced: {created} created, {updated} updated, {skipped} skipped", and the agents appear in the "Catalog" section with the "External" and "From source" marks.

## Source parameters

The source form contains the "Main information", "Catalog resources", and "Connection" tabs.
Collections are set in the source settings and applied to its agents.

### Basic information

The "Main information" tab contains the following fields:

| Name | Required | Description |
| --- | --- | --- |
| "Enable" | — | A disabled source is not synchronized and does not disable agents it has already created |
| "Name" | Yes | Source name |
| "Identifier" | Yes | Unique source ID |
| "Description" | No | Source description |
| "Owner" | No | Owner of agents whose entities have no owner set |
| "Owning team" | No | Owner team of agents whose entities have no team set |

### Catalog resources

On the "Catalog resources" tab, at least one resource binding is set.
A resource can be bound to a source only once.

Each binding has the following fields:

| Name | Required | Description |
| --- | --- | --- |
| "Resource" | Yes | Catalog resource whose entities become agents |
| "Agent Card URL property" | Yes | Resource property containing the link to the agent card |
| "Enabled property" | No | Property that determines whether to register the entity as an agent |
| "Filter rules" | No | Rules that limit the selection of entities |

On save, the portal checks that the resource exists and the specified properties are declared in the resource.

#### Enablement property

If the enablement property is set, the entity is skipped when the property is empty or its value is `false`, `0`, `no`, or `off` in any case.
Any other value is treated as enabled.

#### Filter rules

Filter rules limit which entities of the resource are synchronized as agents.
The rules are applied in order, and each subsequent rule works with the result of the previous one.
If there are no rules, all entities of the resource are synchronized.

Each rule consists of the following fields:

- "Action": "Keep only matches" or "Remove matches".
- "Source": Path to the entity field in Go template format.
- "Condition": Regular expression against which the field value is checked.

In the "Source" field, the entity is available through the `.entity` root:

```go
{{ .entity.slug }}
{{ .entity.name }}
{{ .entity.description }}
{{ .entity.properties.environment }}
```

For example, to synchronize only production environment agents, add a rule with the "Keep only matches" action, the `{{ .entity.properties.environment }}` source, and the `^production$` condition.

An entity excluded by a rule does not get into the selection, and the agent previously created from it is disabled at the next synchronization.

### Connection

The parameters on the "Connection" tab apply to all agents of the source:

| Name | Required | Description |
| --- | --- | --- |
| "API key" | Yes | Key that the portal passes to each agent of the source in the `X-A2A-Api-Key` and `Authorization: Bearer` headers |
| "Call the address from the agent card" | — | If disabled, the portal calls the base URL of the card. Disable it for in-cluster sources, for example, kagent, that publish addresses reachable only from inside the cluster |
| "MCP collections" | No | Collections of each agent of the source. The run receives their tools that the running user is allowed to use; without collections, the agents work without MCP tools |

The API key is stored encrypted and is not displayed after saving.
When editing the source, leave the field empty to keep the current key.

## Agents from a source

For an agent from a source, the portal sets the card URL, the API key, "Call the address from the agent card", and the MCP collections from the source settings at every synchronization.
Change the collections of such an agent in the source.
In the agent connection, you can change the enabled state and the permission to use the Deckhouse Development Portal (DDP) model proxy; skills are assigned to the agent itself.

If the agent needs the DDP model proxy, enable "Allow the DDP model proxy" in the agent connection after the first synchronization, because the source does not change this switch.

If you disable an agent manually, synchronization does not re-enable it; in the synchronization results, it is shown as skipped with the `manually_disabled` reason.
You can delete an agent from a source only when it is disabled, because the next synchronization would recreate an enabled agent.

## Synchronization results

The "Sync now" button returns a result for each agent.
The notification shows the number of created, updated, and skipped agents; all results other than creation and update are counted as skipped.

Reasons why an agent is skipped or requires attention:

| Reason | Meaning |
| --- | --- |
| `card_url_missing` | The card URL property is not filled in for the entity |
| `card_unreachable` | The card of a new agent cannot be read. The agent is created disabled. An agent that is already working is not disabled when its card is unavailable |
| `invalid_slug` | The entity ID is not valid as an agent ID |
| `duplicate_in_payload` | The selection contains several entities with the same ID |
| `manually_disabled` | The agent was disabled manually |
| `ai_agent_remote_link_slug_conflict` | The ID is already taken by an external agent connected manually or by another source |
| `ai_agent_remote_link_slug_reserved` | The `ddp-agent` ID is reserved for the platform agent |

## Disabling and deleting a source

A disabled source is not synchronized. Agents it created remain in their current state.

When a source is deleted, the agents it created are neither deleted nor disabled.
They become regular external agents, all fields of which can be changed manually.
