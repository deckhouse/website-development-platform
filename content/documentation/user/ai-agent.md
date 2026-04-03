---
title: AI Agent
---

{{< alert level="warning" >}}
Experimental Functionality
{{< /alert >}}

The AI agent is an intelligent assistant built into the Deckhouse Development Platform. It answers questions about the platform, analyzes catalog data, and performs various tasks using MCP (Model Context Protocol) tools.
The AI agent uses customizable AI providers to process requests and can work with various language models, including OpenAI GPT, Ollama, and any models accessible through a compatible REST API.

## Configuring AI Providers

### Provider Types

The platform supports three types of AI providers:

1. **OpenAI** — for working with OpenAI models (GPT-4, GPT-3.5, etc.).
1. **Ollama** — for working with local models via Ollama.
1. **Custom** — for working with any custom REST API compatible with the AI agent request/response format.

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

**Important:**
- The key of an existing credential cannot be changed. To change the key, delete the old credential and create a new one.
- The value of an existing credential can be updated by entering a new value.
- Credentials are encrypted when saved and are never transmitted to the web interface after saving.

#### Using Credentials in Headers

Instead of directly specifying the token in the provider headers, use the template syntax:

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

**How it works:**
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
- **Model**: Enter the model name, such as `llama2` or `mistral`. - **URL**: `http://localhost:11434/api/generate` (or your Ollama server address).
- **Method**: `POST`.

1. Click **Save**.

### Creating a Custom Provider

To set up a custom REST API provider:

1. Create a new provider and select the **Custom** type.
1. Fill in the basic fields (URL, method, authorization headers).
1. Click **Save**.

#### Response Field

The **Response Field** field defines the path to the API response JSON element that contains the model response text. This path is used to extract the response from the JSON returned by the provider's API.

**Path Format:**
- Use dot notation to access nested fields. - For arrays, use an index starting with 0.
- This field is optional — if not specified, the system will attempt to find the answer automatically.

**Examples:**

For OpenAI-compatible APIs:

```json
choices.0.message.content
```

This means: take the first element of the `choices` array, then the `message` field, then the `content` field.

For other formats, possible values are `response.text`, `content`, or `answer`.

**How to determine the correct path:**
1. Make a test request to your API.
1. Examine the JSON response structure.
1. Find the field containing the model's response text.
1. Specify the path to this field in dot notation.

#### Request Body Template

The **Request Body Template** field defines the structure of the JSON request that will be sent to the provider's API. You can use variables in the template; they will be automatically replaced with the current values.

**Available variables:**
- `{{.prompt}}` — the user's question text (will be automatically escaped for JSON).
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

