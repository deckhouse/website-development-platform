---
title: MCP server
description: Connect external AI clients to Deckhouse Development Portal and use built-in and collection tools over MCP.
---

{{< alert level="warning" >}}
Experimental feature
{{< /alert >}}

The MCP server is a Deckhouse Development Portal (DDP, portal) component that implements the Model Context Protocol (MCP). It enables external AI clients, such as LM Studio and Claude Desktop, to interact with the portal.
The server uses JSON-RPC 2.0 and provides tools for working with portal resources and proxying requests to external infrastructure services.

MCP is an open protocol for connecting AI models to external systems. For details, refer to the [official MCP website](https://modelcontextprotocol.io/).

{{< alert level="info" >}}
In addition to built-in portal tools, the MCP server can call tools from [MCP collections](../mcp-management/#mcp-collections) available to the user. Configure MCP servers, custom MCP tools, and collections under [MCP management](../mcp-management/).
{{< /alert >}}

## Available tools

The following **built-in** portal tools are available. Tools of the `external` and `custom` types become available after you [synchronize the catalog](../mcp-management/#mcp-servers) and [add them to an MCP collection](../mcp-management/#mcp-collections).

### get_resources

Gets a list of resources.

Parameters: None.

Returns: A list of resources.

Example:

```sh
Get a list of resources
```

---

### get_external_services

Gets a list of external services, such as GitLab and SonarQube.

Parameters: None.

Returns: A list of external services.

Example:

```sh
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

```sh
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

```sh
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

```sh
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

```sh
Get a list of projects from the external GitLab service
```

---

### get_actions

Gets a list of actions.

Parameters: None.

Returns: A list of actions.

Example:

```sh
Get a list of actions
```

---

### get_datasources

Gets a list of data sources.

Parameters: None.

Returns: A list of data sources.

Example:

```sh
Get a list of data sources
```

---

### get_processes

Gets a list of processes.

Parameters: None.

Returns: A list of processes.

Example:

```sh
Get a list of processes
```

## Connecting to the MCP server

### LM Studio

1. Get the connection parameters:

   - Sign in to Deckhouse Development Portal.
   - Get an API token under **Profile**.
   - Note the portal URL, for example, `https://ddp.example.com`.

1. Configure LM Studio:

   - Open LM Studio.
   - Open **Settings**.
   - Find **MCP Servers** or **Model Context Protocol**.
   - Select **Add Server**.

1. Configure the server:

   - **Server Name**: `DDP MCP Server`, or another name.
   - **Server URL**: `https://<DOMAIN>/api/v2/mcp`.
   - **Transport**: `HTTP` or `JSON-RPC`.
   - **Authentication**:
     - **Type**: `Bearer Token` or an equivalent that uses the `Authorization` header.
     - **Header**: `Authorization: Bearer <your_api_token>`.
     - **Token**: Enter the portal API token from **Profile**.

1. Verify the connection:

   - Save the configuration.
   - LM Studio connects to the server after saving.
   - After a successful connection, the following tools are available:
      - `get_resources` — Gets a list of resources.
      - `get_external_services` — Gets a list of external services, such as GitLab and SonarQube.
      - `get_resource_entities` — Gets all entities of the selected resource.
      - `get_entity` — Gets one entity by UUID.
      - `get_entity_relations` — Gets entity relations by resource and identifier.
      - `get_external_data` — Sends an HTTP request to an external service using the user's credentials.
      - `get_actions` — Gets a list of actions.
      - `get_datasources` — Gets a list of data sources.
      - `get_processes` — Gets a list of processes.

After connecting, you can use these tools in conversations with models.

All calls use your access permissions.

### Connecting other MCP clients

The Deckhouse Development Portal MCP server is compatible with any client that supports MCP over JSON-RPC 2.0.

To connect a client:

1. **Endpoint URL**: `https://your-platform.com/api/v2/mcp`.
1. **Protocol**: JSON-RPC 2.0.
1. **Authentication**:
   - Header: `Authorization: Bearer YOUR_API_TOKEN`.
   - `YOUR_API_TOKEN` is your portal API token from **Profile**.
1. **Method**: POST.

### MCP server request example

HTTP headers:

```sh
Authorization: Bearer your-api-token-here
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
      "resource_uuid": "target-resource-uuid"
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
        "text": "[{\"uuid\":\"...\",\"name\":\"Service 1\",\"properties\":{...}},...]"
      }
    ]
  }
}
```

## Security

Authentication:

- Every MCP server request must be authenticated with an API token from **Profile**. Pass the token in the `Authorization: Bearer <api_token>` header.
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
- Verify that you use the `Authorization: Bearer <api_token>` header.

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
