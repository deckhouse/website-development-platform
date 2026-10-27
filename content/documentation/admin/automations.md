---
title: Automations
menuTitle: Automations
description: Automatically running actions, processes, and datasource syncs when entities change.
weight: 70
---

Automations are the Deckhouse Development Portal (DDP) mechanism for responding to changes in entity specifications.

Each automation can subscribe to updates of entities of a particular resource according to specific rules.
When an update event occurs, the automation runs an action, a process, or a datasource sync.

According to its settings, an automation can run for the entity that fired the trigger as well as for entities related to it.

## Automation settings interface

The form for creating and editing an automation is divided into logical sections.

### Triggers

This section configures the conditions under which the automation runs.
Each automation can contain any number of triggers.

For each trigger, you specify a "Resource": the resource whose entity events trigger the automation.
You also specify a "Condition" in [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) format.
It describes which property change triggers the automation and which value the changed property must take.

For example, with the `{{ ne .entity.properties.id 0 }}` condition, the automation runs if the value of the `id` property has changed and is no longer zero.
With the `{{ eq .entity.properties.status "Failed" }}` condition, the automation runs if the value of the `status` property has changed to `Failed` from any other value.

### Execution

This section configures what runs when the automation trigger fires.

In the "Execution type" field, select the type of object to run: an action, a process, or a datasource sync.
In the "Execution object" field, select the specific object (action, process, or datasource).

A process runs only for entities of the resources it is bound to in the "Resource" field of the [process](../processes/overview/#basic-information).
Therefore, an automation process must be bound to all resources for whose entities the automation runs it: to the trigger resources if execution runs for the entity that fired the trigger or on a schedule, and to the resource at the other end of each selected relation if running for related resources is enabled.

If execution runs for the entity that fired the trigger or on a schedule, processes not bound to the trigger resources are inactive in the "Execution object" list and labeled "not bound to the trigger resources". The portal checks the binding to related resources when saving.
An automation with a process that is not bound to the required resources is not saved.
The portal responds with a `422` error with the text "A process runs only for entities of its resources: bind it to the resources for whose entities the automation runs it (trigger resources or related resources), or select another process".

Actions and processes have the additional "Run execution for related resources" setting.
When it is enabled, execution automatically runs for all entities that have the corresponding relation to the changed entity.
In this case, execution does not run for the entity that fired the event.

### Sync

This section configures periodic sync, that is, running the automation automatically on a schedule.

The "Periodic synchronization" parameter enables and disables periodic sync.
The "Launch frequency" parameter takes a cron expression that sets how often the automation runs, for example, `0 0 * * *` to run daily at midnight.

## Triggers

Automation triggers are based on events that are generated every time any entity of any resource is created, changed, or deleted.

Automations do the following:

1. Listen to events of the resources specified in the trigger configuration.
1. Check whether the value of the property specified in the trigger conditions has changed.
1. If it has changed, compare the current property value with the condition.
1. If the condition is met, start execution.

{{< alert level="warning" >}}
New automations process only events that occurred after they were created.
Events that occurred before the automation was created are not processed.
{{< /alert >}}
