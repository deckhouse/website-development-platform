---
title: Agent runs
weight: 50
description: AI agent run history — statuses, run triggers, results, and stopping a run.
---

{{< alert level="warning" >}}
AI agents are a Preview feature.
Agent types, available settings, and the API may change in future releases without backward compatibility.
{{< /alert >}}

Every call to an AI agent creates a run: an answer to a question in the AI assistant, a response of the "AI chat" widget, or the execution of the "AI agent" process element.
A run stores the message sent to the agent, the response, tool calls, the error, and the result.

## Run history

The history is available in "AI" → "Usage" → "Audit", on the "Agent runs" tab, with the `read:ai-agent-runs` permission.
A user sees only their own runs; a super administrator sees runs of all users.

You can filter the run table by the following fields:

| Filter | Description |
| --- | --- |
| "Status" | Run status |
| "Trigger" | Where the agent was started from |
| "Agent" | Agent ID |
| "Period start", "Period end" | Time interval of the run start. The period start must not be later than the period end |
| "Search by agent, provider, or error" | Substring of up to 64 characters in the agent ID or name, the AI provider name, or the error text |

## Run statuses

A run has one of the following statuses:

| Status | Meaning |
| --- | --- |
| "Running" (`running`) | The agent is performing the task |
| "Succeeded" (`succeeded`) | The agent returned a response |
| "Failed" (`failed`) | The run ended with an error, exceeded the maximum duration, or was interrupted |
| "Cancelled" (`cancelled`) | The run was stopped by a user or together with the process |

## Triggers

The "Trigger" field shows where the agent was started from:

| Trigger | Where the agent is started from |
| --- | --- |
| "Chat" | A question in the AI assistant |
| "Widget" | A question in the "AI chat" widget |
| "Process" | The "AI agent" element in a process run |

## Run details

To open the details, click "Open" in the run row.
The "Agent run details" window shows the agent, status, trigger, AI provider, start and end time, the error and the provider response, and the run result.

The result type is shown in the "Deliverable type" field:

| Type | When used | Reference |
| --- | --- | --- |
| "Message" | The agent returned a text response | Run UUID |
| "Process store" | The agent was run by a process element, and the result was written to the process store | Store key |
| "Merge request", "Branch", "Patch" | The external agent reported the result in the response metadata | Link to the merge request, branch, or patch |

For details on how an external agent reports the result type, see ["Run result"](external-agents/#run-result).

## Stopping a run

To stop a run in the "Running" status, click "Stop" in the run row or in the details window and confirm the action.
The button is available with the `cancel:ai-agent-runs` permission.

After the run is stopped:

- The run gets the "Cancelled" status.
- The portal revokes the run token, and the agent can no longer call portal tools or access the model through the portal.
- If the agent was started by a process element, the process continues according to the element's error handling rules.

The portal does not send a task cancellation command to the external agent.
The agent keeps working until its next call to the portal, which fails with a `401` error.

Runs created by process elements are also stopped when the process is stopped or force-restarted.
