---
title: MCP management
description: Connect upstream MCP servers, custom MCP tools, and MCP collections for the AI assistant and external MCP clients.
---

Use **AI** → **MCP** to configure tools that the AI assistant and [built-in MCP server](../mcp-server/) can call on behalf of a user.

Access to **AI** is controlled by the global `view:ai-page` permission. Separate global `read:` and `edit:` permissions control MCP server, tool, and collection configuration. For details, refer to the [role model](../../admin/security/rbac/#global-permissions).

## Overview

| Section | Purpose |
|---------|---------|
| **MCP servers** | Connect upstream MCP servers and synchronize their tool catalogs |
| **MCP collections** | Group tools and control user access |
| **MCP tools** | Create custom MCP tools without an upstream MCP server |
| **Catalog** | View all available tools |

The AI assistant displays the following tool types:

- `internal` — Built-in platform tools described in the [MCP server documentation](../mcp-server/).
- `external` — Tools retrieved from a connected upstream MCP server.
- `custom` — Custom MCP tools.

Tools of the `external` and `custom` types can be called only if they belong to an enabled MCP collection available to the user.

## MCP servers

Connect an upstream MCP server to import its tools into the platform catalog.

1. Go to **AI** → **MCP** → **MCP servers**.
1. Select **Connect**.
1. On the **General information** tab, specify **Name**, **Identifier**, **Description**, **Owner**, and **Team**.
1. On the **Configuration** tab:
   1. Enable the **Enabled** toggle.
   1. Select a **Transport**: `HTTP` or `SSE`.
   1. Specify the upstream MCP server **URL**.
   1. If necessary, add **HTTP headers** and **Credentials** for authentication.
1. Select **Save**.

After saving, open the server card and select **Synchronize** to load the tool catalog. On the **Tools** tab, enable the required tools and add **Tags** if necessary.

## MCP tools

Create a custom MCP tool that the AI assistant calls directly without an upstream MCP server.

1. Go to **AI** → **MCP** → **MCP tools**.
1. Select **Add**.
1. On the **General information** tab, specify **Name**, **Description**, **Owner**, **Team**, and **Tags**.
1. On the **Configuration** tab:
   1. Enable the **Enabled** toggle.
   1. Define the **Argument schema** as a `JSON Schema` with the root type `object`.
   1. If necessary, specify **Path parameter mapping** as a `JSON` object that maps argument names to `{placeholder}` segments in the executor URL.
   1. If necessary, specify **Query parameter mapping** as a `JSON` object that maps argument names to URL query parameter names.
1. On the **Authorization** tab, specify the executor endpoint **URL**, **HTTP method**, **HTTP headers**, and **Credentials**. The URL can contain `{placeholder}` segments for path parameters and credential placeholders such as `{{ .credentials.tenant_id }}`.
1. Select **Save**.

For example, assume the argument schema defines the `space_id` argument, the executor URL is `https://api.example.com/v1/spaces/{spaceId}/boards`, and the path parameter mapping is `{"space_id": "spaceId"}`. If the tool is called with `space_id=42`, the request is sent to `https://api.example.com/v1/spaces/42/boards`.

For `GET` and `DELETE`, arguments not mapped to path parameters are passed only as query parameters. For `POST`, `PUT`, and `PATCH`, arguments not mapped to path or query parameters are passed in the JSON request body.

## MCP collections

An MCP collection groups catalog tools and determines which tools are available to AI assistant users and external MCP clients.

1. Go to **AI** → **MCP** → **MCP collections**.
1. Select **Create**.
1. On the **General information** tab, specify **Name**, **Identifier**, **Description**, **Owner**, and **Team**.
1. On the **Configuration** tab:
   1. Enable the **Enabled** toggle.
   1. Under **Tools**, select tools from the available catalog. Each item includes the MCP server identifier in parentheses.
1. Select **Save**.

To let a user call collection tools, open the collection card menu and select **Configure access**. Assign users or teams a role with the `use:mcp-collections` permission. For the permission list, refer to the [role model](../../admin/security/rbac/#mcp-collections).

The collection owner and super administrator automatically receive access to the collection.

## Catalog

The **Catalog** section displays all tools available to the current user based on MCP collections and access permissions. This section is read-only. Edit tools on the **MCP servers**, **MCP tools**, and **MCP collections** pages.

## Integration with the AI assistant and MCP server

Tools from MCP collections are used as follows:

- In the [AI assistant](../ai-assistant/#mcp-tools), **Available tools** displays built-in tools and tools from available collections.
- The platform [MCP server](../mcp-server/) returns and calls the same collection tools available to the user based on their RBAC permissions.
