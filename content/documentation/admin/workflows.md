---
title: Workflows
menuTitle: Workflows
description: Chaining actions together and running them in parallel or sequentially.
weight: 75
---

Workflows are a mechanism for combining actions into a chain and controlling the execution flow.

Like actions, workflows are bound to specific resources and can be run only for entities of the resource they are bound to.

## Action run type

Actions in a workflow can be run in parallel or sequentially.

If you select a parallel run, all actions start at the same time.
If you select a sequential run, actions start in the order in which they appear in the workflow's action list.
If one of the actions fails, all subsequent actions are created but not started.

{{< alert level="info" >}}
With a sequential run, if any preceding action requires approval, the actions that follow it start as soon as that action enters the `Unapproved` status.
{{< /alert >}}

Each workflow action can be either required or optional.
If an action is required, the user running the workflow cannot disable it.
If an action is optional, the user can disable it when running the workflow, and the action will not start.

## Running a workflow

When running a workflow, the user is shown a form for filling in the run parameters of each action in the workflow.
Optional actions can be disabled in this form.

## Workflow parameters

Configuring and using workflow parameters is described in the ["Templating"](../../user/templating/#workflow-parameters) section.
