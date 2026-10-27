---
title: AI provider templates
weight: 60
description: AI provider templates — preparing a language model connection for portal users, shared and personal tokens, and AI providers of service accounts.
---

{{< alert level="warning" >}}
AI agents are a Preview feature.
Agent types, available settings, and the API may change in future releases without backward compatibility.
{{< /alert >}}

An AI provider template is a language model connection that an administrator prepares for portal users.
The template defines the URL, method, headers, and models, and when connecting a provider from the template, the user selects a model and, depending on the template, enters their own token.

A user needs an AI provider to work with the [platform agent](platform-agent/) and with external agents that are allowed to use the Deckhouse Development Portal (DDP) model proxy.

## Token source

A template works in one of the modes set by the "Token source" field:

| Mode | How it works |
| --- | --- |
| "Personal token of each user" | The user connects a provider from the template in their profile and enters their own token |
| "One shared token for everyone" | The token is set in the template. The provider appears for every user allowed to use the template without connecting it. Such a provider cannot be deleted or changed in the profile |

The token is stored encrypted and is substituted into the request headers in place of `{{ .token }}`.

## Creating a template

To create a template, follow these steps:

1. Go to "AI" → "Agents" → "Provider templates".
   Creating templates is available with the global `edit:ai-provider-templates` permission; viewing and editing existing templates can also be allowed by object-level roles.
1. Click "Create".
1. Fill in the template fields as described in ["Template parameters"](#template-parameters).
1. Click "Save".

To prepare a template in advance, save it disabled.
The disabled state is preserved on creation; enable the template when it is ready for use.

## Template parameters

The template form contains the following fields:

| Name | Required | Description |
| --- | --- | --- |
| "Name" | Yes | Template name that users see when connecting a provider |
| "Description" | No | Short description shown to users when connecting |
| "Default model" | Yes | Model used if the user has not selected another one |
| "Allowed models" | No | Models that the user can select. An empty list means any model |
| "Enabled" | — | A disabled template cannot be used for new connections |
| "URL" | Yes | Provider API URL, for example, `https://api.openai.com/v1/chat/completions` |
| "Method" | Yes | HTTP method of the request to the provider |
| "Token source" | Yes | Template mode. For details, see ["Token source"](#token-source) |
| "Token" | For a shared token | Token used to authorize all users of the template. When editing, an empty value keeps the current token |
| "Headers" | No | HTTP request headers. The token is substituted in place of `{{ .token }}`, for example, `Authorization: Bearer {{ .token }}` |
| "Response field" | No | Path to the model response text in the provider's JSON response, for example, `choices.0.message.content` |
| "Request body template" | No | JSON template of the request body with the `{{.prompt}}` and `{{.model}}` variables |

For the template to work with agents, specify an API compatible with OpenAI Chat Completions: the URL must end with `/v1` or `/v1/chat/completions`.

If you switch a template from a shared token to a personal one, the shared token is deleted.
You cannot delete a template from which users have connected providers.

### OpenAI template example

Example of a template with a shared token for the OpenAI API:

| Field | Value |
| --- | --- |
| "Name" | `OpenAI` |
| "Default model" | `gpt-4o` |
| "URL" | `https://api.openai.com/v1/chat/completions` |
| "Method" | `POST` |
| "Token source" | "One shared token for everyone" |
| "Token" | OpenAI API key |
| "Headers" | `Authorization: Bearer {{ .token }}` |

## Access to a template

Using a template requires the global `use:ai-provider-templates` permission or the object-level `use:ai-provider-template` permission.
Ownership grants permissions when `owner-is-admin` is enabled.
This applies to templates with both personal and shared tokens.

For a template with a shared token, access automatically creates a provider for the user.
For a specific template, grant the object-level `use:ai-provider-template` permission; the global `use:ai-provider-templates` permission grants access to all templates.
To grant access, select "Edit role bindings" in the template card menu and create a role binding for users or teams.

Disabled templates are visible to users with read or edit permissions on the template, granted globally or on the object.
Other users see the template headers only with placeholder values.

## User provider from a template

For a provider connected from a template, the user can change only the name, the model within the "Allowed models" list, and the token.
The portal takes the URL, method, headers, and other connection parameters from the template, so template changes apply immediately to all connected providers.

In the profile, such a provider is marked as "from template", and a provider from a template with a shared token is marked as "shared token".
For details on how a user connects a provider, see ["Connecting an AI provider"](../../user/ai-assistant/#connecting-an-ai-provider).

## AI providers of service accounts

A service account cannot sign in to a profile, so its AI providers are managed by an administrator with the `edit:users` permission.
An AI provider for a service account is needed, for example, when a process with the "AI agent" element runs on behalf of a service account.

To configure an AI provider for a service account, follow these steps:

1. Go to "Administration" → "Users".
1. In the service account row, click the button with the "Manage AI provider for this service account" tooltip.
1. Connect a provider from a template or set your own configuration, as in a user profile.
