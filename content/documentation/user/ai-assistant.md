---
title: AI assistant
description: Configure AI providers, credentials, chats, context, and MCP tools for the AI assistant.
---

{{< alert level="warning" >}}
Experimental feature
{{< /alert >}}

The AI assistant is an intelligent helper built into Deckhouse Development Platform (DDP). It answers questions about the platform, analyzes catalog data, and performs tasks using Model Context Protocol (MCP) tools.

The AI assistant uses configurable AI providers to process requests. It supports various language models, including OpenAI GPT, Ollama, and any models available through a compatible REST API.

## Connecting an AI provider

{{< alert level="info" >}}
Users configure AI providers in their profiles.
{{< /alert >}}

To connect a new AI provider:

1. Open **Profile** → **AI providers**.
1. Select **Add**.
1. Fill in **Name**, **Model**, **URL**, **Method**, and **Headers**. If necessary, specify the [Response field](#response-field) and [Request body template](#request-body-template).
1. When using tokens in headers, store the credentials and insert them through [templating](#templating-in-headers).
1. Select **Save**.

For common API configurations, refer to [Configuration examples](#configuration-examples).

## Provider credentials

The credential system securely stores tokens and keys: it encrypts them in the database and inserts them into request headers through templating.

### Adding credentials

To add credentials:

1. In the provider creation or editing form, select **Manage credentials**.
1. In the dialog, select **Add credentials**.
1. Enter a **Key** and **Value** pair, for example, `api_key` and its secret.
1. Select **Save**.

### Editing credentials

Consider the following:

- You cannot change the key of existing credentials. Delete the entry and create a new one.
- To update a value, enter a new one.
- Saved values are not displayed in the interface.

### Templating in headers

Use a substitution instead of a plaintext token:

```sh
Authorization: Bearer {{ .credentials.api_key }}
```

Here, `Authorization` is the header, `credentials` is the encrypted credential store, and `api_key` is the key defined in the credentials.

## Response field

The **Response field** defines the path to the model response text in the JSON API response body, for example, `choices.0.message.content`. If the field is empty, the platform attempts to locate the response text automatically.

{{< alert level="info" >}}
To determine the path, send a test API request, open the response, and locate the field containing the model response text in the JSON body.
{{< /alert >}}

## Request body template

The **Request body template** field contains the JSON structure sent to the API.

The following variables are available:

- `{{.prompt}}` — User request text.
- `{{.model}}` — Model configured for the provider.

### Structure examples

OpenAI-compatible chat:

```json
{
  "model": "{{.model}}",
  "messages": [
    {
      "role": "user",
      "content": "{{.prompt}}"
    }
  ],
  "temperature": 0.7
}
```

Alternative message format:

```json
{
  "messages": [
    {
      "content": "{{.prompt}}",
      "role": "user"
    }
  ],
  "model": "{{.model}}",
  "stream": false
}
```

Minimal format:

```json
{
  "query": "{{.prompt}}",
  "model_name": "{{.model}}"
}
```

{{< alert level="info" >}}
The template must be valid JSON. You can add fields such as `temperature` and `max_tokens`.
{{< /alert >}}

## Configuration examples

### OpenAI API (Chat Completions)

1. **Name** — Any name, for example, `ChatGPT`.
1. **Model** — For example, `gpt-4` or `gpt-3.5-turbo`.
1. **URL** — `https://api.openai.com/v1/chat/completions`.
1. **Method** — `POST`.
1. **Headers** — For example, `Authorization: Bearer {{ .credentials.openai_api_key }}`. Add the `openai_api_key` key under **Manage credentials**.
1. **Response field** — `choices.0.message.content`.
1. **Body template** — Use the [first example](#structure-examples) under **Request body template**.

### Ollama (`/api/generate`)

1. **URL** — `http://localhost:11434/api/generate`, or your Ollama address.
1. **Method** — `POST`.
1. **Response field** — `response`.
1. **Body template**:

   ```json
   {
     "model": "{{.model}}",
     "prompt": "{{.prompt}}",
     "stream": false
   }
   ```

### Custom REST API

- **URL** — Your service endpoint, for example, `https://api.example.com/v1/chat`.
- **Method** — Usually `POST`.
- **Headers** — For example, `Authorization: Bearer {{ .credentials.api_key }}` and `Content-Type: application/json`.
- **Response field** — Path to the text field in your JSON response.
- **Body template** — Base it on the first example under [Request body template](#request-body-template). Add fields such as `max_tokens` if necessary.

## Using the AI assistant

The assistant panel opens on the right. Use the button in the lower-right corner of the screen to open it.

### Selecting a provider

The provider list is at the top of the chat panel. If only one provider is available, it is selected automatically.

### Chats

Conversations are organized into chats. The chat list and **New chat** are on the left.

Chats have the following constraints:

1. Each user can have up to 20 chats. When you reach the limit, delete old chats from the **⋯** menu.
1. Before the first message, a chat has a default name. After the first question, the question text becomes the chat name. You can select **Rename** to change it.
1. Deleting a chat and its message history is irreversible.

### Sending context

Configure **Send context** separately for each chat:

1. **Enabled** — Sends the conversation context for this chat with the request.
1. **Disabled** — Sends only the current message. This uses fewer tokens but does not retain chat context.

With context enabled, long histories are not sent in full with every request. Recent messages are sent in full, while earlier messages are summarized in a separate request to the same provider.

{{< alert level="warning" >}}
Sending context increases token usage when interacting with the model.
{{< /alert >}}

### MCP tools

The AI assistant uses built-in platform tools and tools from [MCP collections](../mcp-management/#mcp-collections) available to the user.

Expand **Available tools** in the chat panel to view each tool's name, type (`internal`, `external`, or `custom`), arguments, and example. Select an example to insert its text into the input field. The model can call multiple tools in one request.

- Built-in tools (`internal`) and their parameters are described in the [MCP server documentation](../mcp-server/).
- To connect MCP servers, create custom MCP tools, and configure collections, refer to [MCP management](../mcp-management/).
