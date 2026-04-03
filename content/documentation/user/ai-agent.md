---
title: AI Assistant
---

{{< alert level="warning" >}}
Experimental Feature
{{< /alert >}}

The AI ​​Assistant is an intelligent assistant built into the Deckhouse Development Platform. It answers questions about the platform, analyzes catalog data, and performs various tasks using MCP (Model Context Protocol) tools.
The AI ​​Assistant uses customizable AI providers to process requests and can work with various language models, including OpenAI GPT, Ollama, and any models accessible through a compatible REST API.

## Configuring AI Providers

### Provider Types

The platform supports three types of AI providers:

- **OpenAI** — for working with OpenAI models (GPT-4, GPT-3.5, etc.).

- **Ollama** — for working with local models via Ollama.

- **Custom** — for working with any custom REST API compatible with the AI ​​Assistant request/response format.

### Credentials for Providers

A credential system is used to securely store tokens and access keys for AI providers. Credentials are encrypted and stored in the database, and passed to request headers to AI providers via a templating mechanism.

#### Adding Credentials

To add new credentials for AI providers:

1. In the form for creating or editing a provider, click the **Manage Credentials** button.
1. In the dialog that opens, click **Add Credentials**.
1. Enter the key-value pair:
- **Key**: The name of the credential key (e.g., 'api_key', 'openai_token', 'bearer_token').
- **Value**: The token or access key itself.
1. Click **Save**.

#### Changing Credentials

- The key of an existing credential cannot be changed. To change the key, delete the old credential and create a new one.
- The value of an existing credential can be updated by entering a new value.
- Credentials are encrypted when saved and are never transmitted to the web interface after saving.

#### Using Credentials in Headers

Instead of directly specifying the token in the provider headers, use the templating syntax:

```sh
Authorization: Bearer {{ .credentials.api_key }}
```

where `api_key` is the key name you specified when adding the credential.

**Examples:**

Instead of:

```sh
Authorization: Bearer sk-1234567890abcdef
```

Use:

```sh
Authorization: Bearer {{ .credentials.openai_api_key }}
```

**Advantages of using templating:**
- Tokens are not stored in plaintext in the database.
- Credentials can be updated without changing the provider configuration.
- One set of credentials can be used across multiple providers.

**How ​​it works:**
1. You add credentials through the user profile.
2. In the provider headers, specify the template `{{ .credentials.key_name }}`.
3. When executing the request, the system automatically replaces the template with the actual credential value.

### Creating an OpenAI Provider (ChatGPT)

To set up an OpenAI provider, follow these steps:

1. Open your user profile and go to the **AI Providers** tab.
1. Click the **Add** button.
1. Fill out the form:
- **Name**: `ChatGPT` (or any other convenient name).
- **Provider**: Select `OpenAI`.
- **Model**: Enter the model name, for example `gpt-4` or `gpt-3.5-turbo`.
- **URL**: `https://api.openai.com/v1/chat/completions`.
- **Method**: `POST`.
- **Headers**: Add an Authorization header using the credentials:

```sh
Authorization: Bearer {{ .credentials.openai_api_key }}
```

where `openai_api_key` is the name of the credential key (see the [Credentials for Providers](#credentials-for-providers) section).

1. Click **Save**.

### Creating an Ollama Provider

To set up a local Ollama provider:

1. Make sure Ollama is running on your server.
1. Create a new provider:
- **Name**: `Ollama Local`.
- **Provider**: Select `Ollama`.
- **Model**: Enter the model name, such as `llama2` or `mistral`.
- **URL**: `http://localhost:11434/api/generate` (or your Ollama server address).
- **Method**: `POST`.

1. Click **Save**.

### Creating a Custom Provider

To set up a custom REST API provider:

1. Create a new provider and select the **Custom** type.
1. Fill in the basic fields (URL, method, authorization headers).
1. Click **Save**.

#### Response Field

The **Response Field** field defines the path to the API response JSON element that contains the model's response text. This path is used to extract the response from the JSON returned by the provider's API.

**Path Format:**
- Use dot notation to access nested fields.
- For arrays, use an index starting with 0.
- This field is optional — if not specified, the system will attempt to find the response automatically.

**Examples:**

For OpenAI-compatible APIs:

```json
choices.0.message.content
```

This means: take the first element of the `choices` array, then the `message` field, then the `content` field.

For other formats, possible values ​​are `response.text`, `content`, `answer`.

**How ​​to determine the correct path:**
1. Make a test request to your API.
1. Examine the JSON response structure.
1. Locate the field containing the model response text.
1. Specify the path to this field in dot notation.

#### Request Body Template

The **Request Body Template** field defines the structure of the JSON request that will be sent to the provider's API. You can use variables in the template, which will be automatically replaced with the relevant values.

**Available variables:**
- `{{.prompt}}` — the user's question text (will be automatically escaped for JSON).
- `{{.model}}` — the model name specified in the provider's settings.

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
```

For APIs with a minimal request structure:

```json
{
"query": "{{.prompt}}",
"model_name": "{{.model}}"
}
```

#### Template preparation considerations

- The template must be valid JSON.
- The `{{.prompt}}` and `{{.model}}` variables will be automatically replaced when sending the request.
- The `{{.prompt}}` value is automatically escaped for safe insertion into JSON.
- You can add any additional fields required for your API (temperature, max_tokens, etc.).

**Example of a complete custom provider setup:**

- **Name**: `My Custom API`.
- **Provider**: `Custom`.
- **Model**: `my-model-v1`.
- **URL**: `https://api.example.com/v1/chat`.
- **Method**: `POST`.
- **Headers**:

```sh
Authorization: Bearer {{ .credentials.api_key }}
Content-Type: application/json
```

where `api_key` is the name of the credential key (see the [Credentials for Providers](#credentials-for-providers) section).

1. **Response Field**: `choices.0.message.content`. 1. **Request body template**:

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

## Working with the AI ​​Assistant

### Opening Chat

The AI ​​Assistant is accessible through the chat sidebar on the right side of the interface. To open the chat, click the chat button in the lower right corner of the screen.

### Selecting a Provider

At the top of the chat is a provider selector. Select the desired provider from the list of available ones. If you only have one provider configured, it will be selected automatically.

### Available Tools (MCP Tools)

The AI ​​Assistant uses MCP (Model Context Protocol) tools to work with the platform. A full list with descriptions, parameters, and examples is provided in the [MCP server documentation](../user/mcp-server/).

A model can call multiple tools in a single request if necessary.

Key Features:
- Retrieving and analyzing data from the platform catalog.
- MCP proxy to external services (GitLab, SonarQube, Kubernetes, etc.) via `get_external_data` — requests are made with user credentials.

### Viewing Available Tools

The chat displays a list of all available tools with their descriptions and usage examples. You can:

- Expand the **Available Tools** section to view the list.
- Expand each tool to view its parameters and examples.
- Click on an example to paste it into the input field and immediately execute or edit the request.

### Features

- **Data Analysis**: The AI ​​assistant doesn't just return raw data; it analyzes it and provides structured responses.
- **Filtering**: You can request data with conditions, and the assistant will filter it for you.
- **Aggregation**: The assistant can count, group data, and provide statistics.

### Debugging

If the assistant's answer seems incorrect, you can view the debug logs by clicking the question mark icon next to the answer. This will help you understand which tools were called and what data was retrieved.

## Usage Examples

### Example 1: Service Analysis

**Question:**

```sh
Get all services and show their name and creation date
```

**Result:**
The assistant uses MCP tools to retrieve service data and returns a list with their name and creation date.

### Example 2: Action List

**Question:**

```sh
Get a list of actions
```

**Result:**
The assistant will call the MCP tool and return a list of available platform actions.

## Recommendations

- **Use specific questions**: The more specific the question, the more accurate the answer will be.
- **Specify parameters explicitly**: If you know the name of a resource or parameter, specify it in the question.
- **Experiment**: The AI ​​assistant understands natural language, so try different question formulations.
