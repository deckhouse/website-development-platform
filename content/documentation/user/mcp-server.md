---
title: MCP Server
---

{{< alert level="warning" >}}
Experimental Functionality
{{< /alert >}}

The MCP Server is a component of the Deckhouse Development Platform that implements the MCP (Model Context Protocol) and enables interaction between external AI clients (such as LM Studio, Claude Desktop, and others) and platform data.
The server operates over JSON-RPC 2.0 and provides a set of tools for working with platform resources and proxying requests to external infrastructure services.

MCP is an open protocol for interacting with AI models and external systems. More information about the protocol can be found on the [official MCP website](https://modelcontextprotocol.io/).

## Available Tools

### get_resources

Get a list of resources.

**Parameters:** none.

**Returns:** a list of resources.

**Example:**

```sh
Get a list of resources
```

---

### get_external_services

Get a list of external services (GitLab, SonarQube, etc.).

**Parameters:** none.

**Returns:** a list of external services.

**Example:**

```sh
Get a list of external services
```

### get_resource_entities

Get all entities of the selected resource.

**Parameters:**

| Name | Type | Required | Description |
|------------------|--------|------------|-----------------|
| `resource_uuid` | string | yes | Resource UUID. |

**Returns:** a list of resource entities.

**Example:**

```sh
Get all services and show their name and creation date
```

---

### get_external_data

Make an HTTP request to an external service with user credentials.

**Parameters:**

| Name | Type | Required | Description |
|---------------------------|--------|-------------|---------------------------------------------------------------------------|
| `external_service_uuid` | string | yes | The UUID of the external service. |
| `query` | string | yes | Request description (e.g.: "get pipelines for project 123"). |
| `api_path` | string | yes | API path with query parameters (e.g., pagination). |
| `method` | string | no | HTTP method. Default: `GET`. |
| `body` | string | no | Request body for POST/PUT/PATCH (JSON string). |

Credentials and headers are taken from the external service settings in the platform.

**Returns:** the result of the HTTP request to the external service.

**Example:**

```sh
Get a list of projects from the external GitLab service.
```

---

### get_actions

Get a list of actions.

**Parameters:** none.

**Example:**

```sh
Get a list of actions.
```

---

### get_datasources

Get a list of data sources.

**Parameters:** none.

**Example:**

```sh
Get a list of data sources
```

---

### get_processes

Get a list of processes.

**Parameters:** none.

**Example:**

```sh
Get a list of processes
```

## Connecting an MCP Server

### LM Studio

1. **Getting Parameters**

- Log in to Deckhouse Development Platform.
- Obtain your API token in the "Profile" section.
- Write down your platform URL (e.g.: `https://ddp.example.com`).

1. **Setting Up in LM Studio**

- Open LM Studio.
- Go to Settings.
- Find the **MCP Servers** or **Model Context Protocol** section.
- Click **Add Server** or **Add Server**.

1. **Server Configuration**

Fill in the following parameters:

- **Server Name**: `DDP MCP Server` (or any convenient name).
- **Server URL**: `https://ddp.example.com/api/v2/mcp`.
- Replace `ddp.example.com` with the URL of your platform.
- **Transport**: `HTTP` or `JSON-RPC`.
- **Authentication**:
- **Type**: `Bearer Token` or equivalent (`Authorization` header).
- **Header**: `Authorization: Bearer <your_api_token>`.
- **Token**: Paste your platform's API token (from the "Profile" section).

1. **Connection Test**

- Save the configuration.
- LM Studio will attempt to connect to the server.
- If the connection is successful, you will see a list of available tools:
- `get_resources` — Get a list of resources.
- `get_external_services` — Get a list of external services (GitLab, SonarQube, etc.).
- `get_resource_entities` — Get all entities of the selected resource.
- `get_external_data` — Make an HTTP request to an external service with user credentials.
- `get_actions` — Get a list of actions.
- `get_datasources` — Get a list of data sources.
- `get_processes` — Get a list of processes.

After a successful connection, you can use the tools in model dialogs.

All calls will be made with your access rights.

### Connecting in other MCP clients

The Deckhouse Development Platform MCP server is compatible with any clients that support the MCP protocol via JSON-RPC 2.0.

General connection steps:

1. **Endpoint URL**: `https://your-platform.com/api/v2/mcp`.
1. **Protocol**: JSON-RPC 2.0.
1. **Authentication**:
- Header: `Authorization: Bearer YOUR_API_TOKEN`.
- Where `YOUR_API_TOKEN` is your API token from the platform (found in the "Profile" section).
1. **Method**: POST.

### Example request to an MCP server

**HTTP headers:**

```sh
Authorization: Bearer your-api-token-here
Content-Type: application/json
```

**Request body:**

```json
{
"jsonrpc": "2.0",
"id": 1,
"method": "tools/call",
"params": {
"name": "get_resource_entities",
"arguments": {
"resource_uuid": "required-resource-uuid"
}
}
}
```

### Example response from an MCP server

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

### Authentication

- All requests to the MCP server must be authenticated using the API token from the "Profile" section. The token is passed in the `Authorization: Bearer <api_token>` header.
- Access rights correspond to your user rights in the platform.

### Access Rights

- Tools operate with the same rights as the user.
- If you do not have access to a resource, the tool will return an access error.
- Data is filtered according to your RBAC rights.

## Troubleshooting

### Unable to connect to the server

- Check the URL (must end in '/api/v2/mcp').
- Check that the API token is valid.
- Check that the platform is accessible from your computer.
- Check your firewall and proxy settings.

### Authentication error

- Check the token format.
- Check that the token has not expired.
- Check that the 'Authorization: Bearer <api_token>' header is used.

### The tool returns an access error

- Check that your user has permission to access the requested resource.
- Check that the resource name or ID is correct.
- Contact your platform administrator to verify access permissions.

### No data returned

- Check the request parameters.
- Check that the resource exists and contains entities.
- Check the platform logs for detailed error information.
