---
title: AI Agent
---

This means: take the first element of the `choices` array, then the `message` field, then the `content` field.

For other formats, possible values ​​are `response.text`, `content`, or `answer`.

**How ​​to determine the correct path:**
1. Make a test request to your API.
1. Examine the structure of the JSON response.
1. Find the field containing the model's response text.
1. Specify the path to this field using dot notation.

#### Request Body Template

The **Request Body Template** field defines the structure of the JSON request that will be sent to the provider's API. You can use variables in the template, which will be automatically replaced with the relevant values.

**Available variables:**
- `{{.prompt}}` — the text of the user's question (will be automatically escaped for JSON).
- `{{.model}}` — the model name specified in the provider settings.

**Template examples:**

For OpenAI-compatible APIs:

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

For APIs with a different format:

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
``

For APIs with a minimal request structure:

```json
{
"query": "{{.prompt}}",
"model_name": "{{.model}}"
}
```

**Important:**
- The template must be valid JSON. - The `{{.prompt}}` and `{{.model}}` variables will be automatically replaced when sending the request.
- The `{{.prompt}}` value is automatically escaped for safe insertion into JSON.
- You can add any additional fields required for your API (temperature, max_tokens, etc.).

**Example of a complete custom provider setup:**

1. **Name**: `My Custom API`.
1. **Provider**: `Custom`.
1. **Model**: `my-model-v1`.
1. **URL**: `https://api.example.com/v1/chat`.
1. **Method**: `POST`. 1. **Headers**:

```sh
Authorization: Bearer {{ .credentials.api_key }}
Content-Type: application/json
```

Where `api_key` is the name of the credential key (see the [Credentials for Providers](#credentials-for-providers) section).

1. **Response Field**: `choices.0.message.content`.
1. **Request body template**:

```json
{
"model": "{{.model}}",
"messages": [
{
"role": "user",
"content": "{{.prompt}}"
}
],
"temperature": 0.7,
"max_tokens": 1000
}
```

## Working with the AI ​​Agent

### Opening Chat

The AI ​​Agent is accessible through the chat sidebar on the right side of the interface. To open the chat, click the chat button in the lower right corner of the screen.

### Selecting a Provider

At the top of the chat is a provider selector. Select the desired provider from the list of available ones. If you only have one provider configured, it will be selected automatically.

### Available Tools (MCP Tools)

The AI ​​Agent uses MCP (Model Context Protocol) tools to work with the platform. A complete list with descriptions, parameters, and examples is provided in the [MCP server documentation](../user/mcp-server/).

The model can call multiple tools in a single request if needed.

Key features:
- Retrieving and analyzing data from the platform catalog.
- MCP proxy to external services (GitLab, SonarQube, Kubernetes, etc.) via `get_external_data` — requests are executed with user credentials.

### Viewing Available Tools

The chat displays a list of all available tools with their descriptions and usage examples. You can:

- Expand the **Available Tools** section to view the list.
- Expand each tool to view its parameters and examples.
- Click on an example to paste it into the input field and immediately execute or edit the request.

### Features

1. **Data Analysis**: The AI ​​agent doesn't just return raw data; it analyzes it and provides structured responses.
1. **Filtering**: You can request data with conditions, and the agent will perform the filtering.
1. **Aggregation**: The agent can count, group data, and provide statistics.

### Debugging

If the agent's response seems incorrect, you can view the debug logs by clicking the question mark icon next to the response. This will help you understand which tools were called and what data was retrieved.

## Usage Examples

### Example 1: Service Analysis

**Question:**

```sh
Get all services and show their name and creation date
```

**Result:**
The agent uses MCP tools to retrieve service data and returns a list with their name and creation date.

### Example 2: Action List

**Question:**

```sh
Get a list of actions
```

**Result:**
The agent will call the MCP tool and return a list of available platform actions.

## Recommendations

1. **Use specific questions**: The more specific the question, the more accurate the answer.
1. **Specify parameters explicitly**: If you know the name of a resource or parameter, specify it in the question.
1. **Experiment**: The AI ​​agent understands natural language, so try different question formulations.
