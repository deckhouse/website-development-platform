---
title: MCP server
description: Connect external AI clients to Deckhouse Development Portal and use built-in and collection tools over MCP.
---

{{< alert level="warning" >}}
The MCP server is a Preview feature. The set of tools and the API may change in future releases without backward compatibility.
{{< /alert >}}

The MCP server is a Deckhouse Development Portal component that implements the Model Context Protocol (MCP). It enables external AI clients, such as LM Studio and Claude Desktop, to interact with the portal.
The server uses JSON-RPC 2.0 and provides tools for working with portal resources and proxying requests to external infrastructure services.

MCP is an open protocol for connecting AI models to external systems. For details, refer to the [official MCP website](https://modelcontextprotocol.io/).

{{< alert level="info" >}}
In addition to built-in portal tools, the MCP server can call tools from [MCP collections](../mcp-management/#mcp-collections) available to the user. Configure MCP servers, custom MCP tools, and collections under [MCP management](../mcp-management/).
{{< /alert >}}

## Available tools

The following **built-in** portal tools are available. Tools of the `external` type become available after you [synchronize the MCP server catalog](../mcp-management/#mcp-servers) and [add the tools to an MCP collection](../mcp-management/#mcp-collections). Tools of the `custom` type are [created manually](../mcp-management/#mcp-tools) under **MCP tools** and become available after you add them to an MCP collection.

### Tool names

In an MCP client, use the name returned by `tools/list`. The platform agent and OpenCode workspace expose the same tools to the model with the `ddp_` prefix. When configuring a collection, use the name without this prefix:

| Purpose | MCP client and collection catalog | Name for the agent model |
| --- | --- | --- |
| API operation | `get_resources` | `ddp_get_resources` |
| Catalog browser | `query_catalog` | `ddp_query_catalog` |
| Specialized process run | `curated_run_process` | `ddp_curated_run_process` |
| Skill reading | `curated_get_skill` | `ddp_curated_get_skill` |

The names in the second column are used by MCP clients and collection catalogs. Add `ddp_` to names in platform-agent instructions; an external agent may use its own model-facing names.

### Catalog tools

Tools in the **Catalog browser** group search and count entities, retrieve property schemas and relation graphs, inspect process runs, and check process readiness. These tools only read data; readiness checks do not run a process.

The following names are used in the collection catalog and external MCP clients. The platform-agent model and OpenCode receive the `ddp_` prefix, for example `ddp_query_catalog`.

| Tool | Purpose |
| --- | --- |
| `describe_me` | Current user, teams, roles, permissions, time and time zone |
| `get_catalog_schema` | Visible resources, properties and relation types; with `resource`, details of one resource |
| `search_catalog` | Search resources, teams and entities by words |
| `query_catalog` | Filter, count and group entities |
| `get_entity_summary` | Entity card with properties, checks, relations and recent runs |
| `get_entity_relation_graph` | Traverse relations with a depth and path from the source entity |
| `list_process_runs` | Search, count and group process runs |
| `get_process_run` | Diagnose a run using events from the current attempt: errors, approvals, retries and timers |
| `find_operations` | Processes and actions available for reading and linked to an entity resource |
| `check_process_readiness` | Check process parameters and conditions without running it |

### Skills

Skill-reading tools pass instructions and additional files:

| Tool | Arguments | Result |
| --- | --- | --- |
| `curated_list_skills` | Optional string `q` | Skills available to the user with descriptions and latest versions |
| `curated_get_skill` | Required `name`, optional `version` | `SKILL.md` text, version and file list |
| `curated_get_skill_file` | Required `name` and `path`, optional `version` | Text of an additional file |

In an MCP client, select a version by its label. If no label is specified, the latest active version is used. During an agent run, access is limited to the skills assigned to the agent and their pinned versions. `curated_get_skill_file` cannot read binary files or files larger than 1 MiB.

### Workspaces

Workspace tools can prepare a [workspace](workspaces/) from a chat:

| Tool | Arguments | Result |
| --- | --- | --- |
| `curated_get_workspace_templates` | None | Available templates with an active version, repository settings and parameters |
| `curated_create_workspace` | Required `template_uuid`, `name`, `ttl_hours`; optional `provider`, `repository_url`, `base_ref`, `entity_uuid`, `parameters` | Created workspace state; the operation runs asynchronously |
| `curated_get_workspace` | Required `workspace_uuid`, optional `wait_seconds` up to 45 | Status, readiness, last error and active operation |
| `curated_attach_workspace` | Required `workspace_uuid` | Attach the workspace to the current chat |

You can omit `provider` when a suitable provider is already set in the chat context. Otherwise, specify it explicitly. `parameters` is an object containing template variable values, and `ttl_hours` is an integer from 1 to 720.

`curated_attach_workspace` works only in the context of an agent responding in a portal chat. Get the templates, create a workspace, wait for it to become ready, and attach it to the chat.

### get_resources

Gets a list of resources.

Parameters: None.

Returns: A list of resources.

Example:

```text
Get a list of resources
```

---

### get_external_services

Gets a list of external services, such as GitLab and SonarQube.

Parameters: None.

Returns: A list of external services.

Example:

```text
Get a list of external services
```

---

### get_resource_entities

Gets all entities of the selected resource.

Parameters:

| Name            | Type   | Required | Description   |
|-----------------|--------|----------|---------------|
| `resource_uuid` | String | Yes      | Resource UUID |

Returns: A list of resource entities.

Example:

```text
Get all services and show their names and creation dates
```

---

### get_entity

Gets one entity by UUID.

Parameters:

| Name          | Type   | Required | Description |
|---------------|--------|----------|-------------|
| `entity_uuid` | String | Yes      | Entity UUID |

Returns: Data for one entity.

Example:

```text
Get the entity with UUID 3fa85f64-5717-4562-b3fc-2c963f66afa6
```

---

### get_entity_relations

Gets entity relations.

Parameters:

| Name            | Type   | Required | Description       |
|-----------------|--------|----------|-------------------|
| `resource_uuid` | String | Yes      | Resource UUID     |
| `entity_slug`   | String | Yes      | Entity identifier |

Returns: A list of entity relations.

Example:

```text
Get the relations of the "api-gateway" entity in the "Services" resource
```

---

### get_external_data

Sends an HTTP request to an external service using the user's credentials.

Parameters:

| Name                    | Type   | Required | Description                                                                    |
|-------------------------|--------|----------|--------------------------------------------------------------------------------|
| `external_service_uuid` | String | Yes      | External service UUID                                                          |
| `query`                 | String | Yes      | Request description, for example, "get pipelines for project 123"              |
| `api_path`              | String | Yes      | API path with request parameters, such as pagination                            |
| `method`                | String | No       | HTTP method. Default: `GET`                                                     |
| `body`                  | String | No       | Request body for POST, PUT, or PATCH as a JSON string                           |

Credentials and headers are taken from the external service settings in the portal.

Returns: The result of the HTTP request to the external service.

Example:

```text
Get a list of projects from the external GitLab service
```

---

### get_actions

Gets a list of actions.

Parameters: None.

Returns: A list of actions.

Example:

```text
Get a list of actions
```

---

### get_datasources

Gets a list of data sources.

Parameters: None.

Returns: A list of data sources.

Example:

```text
Get a list of data sources
```

---

### get_processes

Gets a list of processes.

Parameters: None.

Returns: A list of processes.

Example:

```text
Get a list of processes
```

## Connecting to the MCP server

### LM Studio

LM Studio 0.3.17 and later supports remote MCP servers. Servers are added in the `mcp.json` file.

To connect LM Studio to the portal MCP server:

1. In the portal, open **Profile** and create an API token.
1. In LM Studio, open the **Program** tab in the right sidebar and select **Install** → **Edit mcp.json**.
1. Add the portal MCP server to the `mcpServers` object:

   ```json
   {
     "mcpServers": {
       "ddp": {
         "url": "https://<DOMAIN>/api/v2/mcp",
         "headers": {
           "Authorization": "Bearer <API_TOKEN>"
         }
       }
     }
   }
   ```

   Where:

   - `<DOMAIN>` is the portal domain;
   - `<API_TOKEN>` is the portal API token.

1. Save the file.

After connecting, the model can call the built-in tools listed in [Available tools](#available-tools) and the tools from MCP collections available to you. All calls use your access permissions.

### Connecting other MCP clients

The MCP server accepts JSON-RPC 2.0 requests over HTTP. To connect a client, use the following parameters:

- Endpoint URL: `https://<DOMAIN>/api/v2/mcp`, where `<DOMAIN>` is the portal domain.
- HTTP method: `POST`.
- Authentication: the `Authorization: Bearer <API_TOKEN>` header, where `<API_TOKEN>` is your portal API token from **Profile**.

The server supports the following JSON-RPC methods:

- `initialize` — returns the server information and the protocol version `2024-11-05`.
- `tools/list` — returns the list of tools available to the user.
- `tools/call` — calls a tool.

Other methods return the `-32601` (`Method not found`) error.

### MCP server request example

HTTP headers:

```text
Authorization: Bearer <API_TOKEN>
Content-Type: application/json
```

Request body:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_resource_entities",
    "arguments": {
      "resource_uuid": "<RESOURCE_UUID>"
    }
  }
}
```

### MCP server response example

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\"status\":200,\"contentType\":\"application/json; charset=utf-8\",\"encoding\":\"json\",\"body\":{\"response\":[],\"error\":null}}"
      }
    ]
  }
}
```

## Security

Authentication:

- Every MCP server request must be authenticated with an API token from **Profile**. Pass the token in the `Authorization: Bearer <API_TOKEN>` header.
- Access permissions match your portal user permissions.

Access permissions:

- Tools use the same permissions as the user.
- If you cannot access a resource, the tool returns an access error.
- Data is filtered according to your RBAC permissions.

## Troubleshooting

### Cannot connect to the server

If you cannot connect to the server:

- Verify that the URL is correct and ends with `/api/v2/mcp`.
- Verify that the API token is valid.
- Verify that the portal is accessible from your computer.
- Check the firewall and proxy settings.

### Authentication error

If authentication fails:

- Verify the token format.
- Verify that the token has not expired.
- Verify that you use the `Authorization: Bearer <API_TOKEN>` header.

### Tool returns an access error

If a tool returns an access error:

- Verify that your user has permission to access the requested resource.
- Verify the resource name or identifier.
- Ask the portal administrator to verify your access permissions.

### No data returned

If no data is returned:

- Verify the request parameters.
- Verify that the resource exists and contains entities.
- Check the portal logs for detailed error information.
