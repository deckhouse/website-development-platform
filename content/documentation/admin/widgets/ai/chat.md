---
title: AI chat
description: Configuration and usage of the language-model chat widget
weight: 10
---

The AI chat widget sends requests to a language model through the selected AI provider. You can configure general instructions (the Global prompt) and a set of quick-question buttons (Quick questions).

## Configuration

| Name            | Required | Description                                                                                                                           |
| --------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Global prompt   | No       | General instructions for each request. The widget combines them with the prompt of the selected quick question before sending it      |
| Quick questions | Yes      | A set of buttons with labels and prompt text (up to 20). Add at least one question for the widget to work correctly                    |

Configure the following fields for each quick question:

| Name          | Required | Description                                                                                       |
| ------------- | -------- | ------------------------------------------------------------------------------------------------- |
| Question name | Yes      | A short label displayed in the widget's bottom panel and in the chat                              |
| Prompt        | Yes      | Instructions sent to the model when the user selects the button. Go templating is supported       |

When writing a prompt, explicitly specify the names of the Model Context Protocol (MCP) tools that the model must call to prepare the response.

Example prompt for a quick question:

```text
1. Call the MCP tool get_external_data for the external service "Deckhouse Code" and retrieve pipelines for the project with ID {{ .entity.properties.deckhouse_code_id }}.
2. Display a table with the 10 latest pipelines.
```

## Using the widget

To use the widget, add at least one [AI provider](../../../../user/ai-assistant/#connecting-an-ai-provider).

The chat does not retain history:

- It displays only one response to the most recent question.
- The response is not retained when the user navigates to another page or refreshes the page.

Before sending a question, the user can customize the prompt by selecting **Send with prompt changes** from the question button menu.
