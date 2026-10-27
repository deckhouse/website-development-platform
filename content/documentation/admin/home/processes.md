---
title: Home page processes
weight: 40
description: Requirements for processes run from the DDP Home page — the run entity, the run context and its keys for each scenario, result display, and naming conventions.
---

Regular catalog processes are run from the Home page.
The Home page passes the process the entity it runs for and the run context — data about the objects selected on the screen.
The process reads the context in templates of parameters, actions, and entity update rules.

Processes for the Home page are created the same way as any other process: [in the process editor](../processes/overview/).
The process is then added to an action slot or template action in the [mapping rules](mapping/#action-slots).

## Running a process from the Home page

All Home page scenarios end with the "Run process" window.
The window title displays the name of the entity the process is run for.

The run window works the same way as when running a process from the catalog:

- If the slot contains several processes, the user selects a process from the list; if there is only one process, it is selected automatically.
- By default, only process parameters marked to be shown in the simplified form and unfilled required parameters are displayed.
  The "All process parameters" toggle shows all parameters and the task configuration.
- Default parameter values are displayed as is, including templates such as `{{ .context.environment.slug }}`.
  Templates are evaluated when the process is run.

After you click "Run", the notification "Process '{name}' started" is displayed.

If a required parameter is still empty after the template is evaluated, for example, because the key is missing from the context, the process does not run, and the "Failed to run process" error is displayed.
If the user does not have the `control:processes` permission, the "Forbidden by RBAC" error is displayed.

## Run context

The run context is available in process templates through the `.context` root: in default values of process parameters, in action bodies, in entity creation and update rules, in gateway conditions, and in loops.

Entities in the context are passed in a single format:

```yaml
name: <ENTITY_NAME>
slug: <ENTITY_SLUG>
uuid: <ENTITY_UUID>
properties:
  <PARAMETER_ID>: <VALUE>
```

Where:

- `<ENTITY_NAME>`: Entity name.
- `<ENTITY_SLUG>`: Entity ID.
- `<ENTITY_UUID>`: Entity UUID.
- `<PARAMETER_ID>`: Parameter ID.
- `<VALUE>`: Parameter value.

The `properties` keys are the IDs of entity properties in the catalog, not mapping roles.
For example, the environment domain in the canonical model is read as `.context.environment.properties.domain`.

Examples of accessing the context:

```go-html-template
{{ .context.environment.slug }}
{{ .context.microservice.name }}
{{ with .context.environment.properties.domain }}{{ $.context.microservice.slug }}.{{ . }}{{ end }}
```

In process parameter values, a missing context key evaluates to an empty string, while in action bodies and conditions it fails the step with an error.
Therefore, read the context in default values of process parameters, and use process parameters in actions.
For details, see [Templating](../../user/templating/#process-launch-context).

## Entity and context by scenario

The table lists the Home page scenarios, the run entity (`.entity`), and the context keys:

| Scenario | Slot | Run entity | `.context` keys |
| --- | --- | --- | --- |
| Creating a microservice | `create_microservice` template action | Selected system | `microservice`, `repository`, `system`, `template` |
| Upgrading a microservice from a template | `upgrade_service_from_template` template action | Microservice | `template`, `upgrade`, `review` |
| Deploying a microservice to an environment | `deploy_microservice` or a template action | Microservice | `environment`, `microservice`, `system` |
| Deleting a microservice from an environment | `delete_microservice` or a template action | Microservice | `environment`, `microservice`, `system` |
| Ordering a resource | `order_resource` | Selected system | `environment`, `system` |
| Deleting a shared resource | `delete_shared_resource` | Shared resource | `system` |
| Connecting and disconnecting a shared resource | `connect_shared_resource`, `disconnect_shared_resource` | Shared resource | `deployment`, `environment`, `system`, `microservice` |
| Creating an environment | `create_environment` | The first environment by name; if there are no environments, the first system by name | Not passed |
| Deleting an environment | `delete_environment` | Environment | Not passed |

The process must be bound to the resource of the run entity from the table.
For example, ordering a shared resource requires binding to the systems resource, deleting it requires binding to the shared resources resource, and creating an environment requires binding to the environments and systems resources.
For existing configurations, check these bindings before running; a resource mismatch causes the run to be rejected.

The `system` key is passed if a system is selected on the screen.
When connecting and disconnecting a shared resource, the `microservice` key is passed if a microservice is found for the deployment.

When creating an environment, the run entity is not related to the environment being created: `.entity` in such a process points to an existing environment or system.
The process gets the new environment's data from its own parameters, which the user fills in.

When deleting a shared resource in the "Resource ordering" section, the `system` key contains the system selected in the section header.

### Creating a microservice

The "Create microservice" wizard passes the process a context with the following structure:

| Path | Value |
| --- | --- |
| `.context.microservice.name` | Microservice name |
| `.context.microservice.slug` | Microservice ID |
| `.context.repository.default_branch` | Value of the template registry's "Default branch" setting; if the setting is empty, `main` |
| `.context.repository.description` | Repository description; can be empty |
| `.context.repository.namespace` | Selected namespace in the version control system: `id` (the `vcs_namespace_id` role value), `name`, `slug`, `uuid` |
| `.context.system` | Selected system in the entity format |
| `.context.template.id` | Template ID |
| `.context.template.version` | Selected template version |
| `.context.template.source` | Name of the registry the template was obtained from |
| `.context.template.language` | Template language |
| `.context.template.values` | Template parameter values from the "Parameters" step. Parameters with the `x-ddp-from` annotation are not passed |

Template parameters with the `x-ddp-from` annotation are filled in from the same context when the repository is created.
For details, see [Template pack](../templates/packs/#binding-parameters-to-the-launch-context).

### Upgrading a microservice from a template

The "Upgrade" button on the microservice card and automatic upgrades pass the same context to the process:

| Path | Value |
| --- | --- |
| `.context.template.id` | Microservice template ID |
| `.context.template.version` | Target template version — the latest available one |
| `.context.upgrade.strategy` | Upgrade strategy: the value of the microservice's `microservice_binding_template_upgrade_strategy` role if set; otherwise, the strategy from the template pack |
| `.context.upgrade.overwrite` | Paths to overwrite: the microservice or pack value |
| `.context.upgrade.add_if_missing` | Paths to add: the microservice or pack value |
| `.context.review.title` | Merge request title: `Upgrade <TEMPLATE_ID> to <VERSION>` |

How the upgrade is performed is described in [Upgrading from a template](../templates/updates/).

## Displaying the result

The Home page tracks running processes and displays their state while the process is running.
Process completion notifications are not displayed; the user sees the outcome in the "My activities" panel.

While a process is running:

- The top of the Home page displays the line "Processes running: {count}"; for processes awaiting approval — "Awaiting approval: {count}"; for processes of other users — "Launched by: {initiators}".
- The row or card of the object the process is running on is marked with an indicator and the "running" caption, and its action buttons are unavailable.
- For an object being created, a temporary row or card with the "creating" caption is displayed.

The Home page checks the state of runs at intervals of 2–15 seconds.
Tracking stops when the process completes, when the process timeout expires, or, if no timeout is set, after 30 minutes.
Paused processes and processes awaiting approval are tracked without a time limit.
The state of runs is loaded from the server, so indicators persist after a page reload and are visible to other users.

After the process completes, the "Systems", "Microservices", "System deployment", "Resource ordering", and "Environments" sections reload their data, and the temporary row is replaced with the created object.

### Process conventions

The temporary row is replaced with the created object only if the object has appeared in the catalog and matches the expected one.
For the result to be displayed immediately after the process completes, meet the following conditions:

- Add a datasource synchronization or entity creation task at the end of the process.
  Otherwise, the object appears on the Home page only after the next datasource synchronization.
- In an environment creation process, name the environment ID parameter `slug`.
  The Home page uses the value of this parameter to build the temporary row and find the created environment.
- In a microservice deployment process, create a deployment with the same name as the microservice.
- In a resource ordering process, suggest a resource name in the `<SYSTEM_ID>-<ENVIRONMENT_ID>` format.
  If the user changes the name, the temporary row disappears when the process completes.

## Dataset processes

The "Canonical architecture" dataset and related datasets create ready-made processes for Home page slots.
You can use them as a model for your own processes.

| Process | Dataset | Slot | Context and parameters |
| --- | --- | --- | --- |
| "Create microservice from template" | "Microservice template registry" | `create_microservice` template action | Tasks "Create GitLab project", "Create repository from template", "Register microservice entity". The "GitLab group" parameter — `{{ with .context.repository.namespace }}{{ .id }}{{ end }}` |
| "Upgrade microservice from template" | "Microservice template registry" | `upgrade_service_from_template` template action | Tasks "Update repository from template" and "Create code review" |
| "Deploy microservice to an environment" | "Microservice template registry" | `deploy_microservice` template action | The `environment` parameter — `{{ .context.environment.slug }}`. Ingress host — `<MICROSERVICE_ID>.<ENVIRONMENT_DOMAIN>` |
| "Delete microservice from an environment" | "Microservice template registry" | `delete_microservice` template action | — |
| "Create environment" | "Environment management" | `create_environment` | Parameters "Identifier" (`slug`) and "TTL" |
| "Delete environment" | "Environment management" | `delete_environment` | The "Environment ID" parameter — `{{ .entity.slug }}` |
| "Connect secrets to application", "Disconnect shared resource" | "Shared resource binding" | `connect_shared_resource`, `disconnect_shared_resource` | — |
| Processes for ordering and deleting managed services | "Deckhouse managed PostgreSQL", "Deckhouse managed Valkey", "Deckhouse managed Kafka" | `order_resource`, `delete_shared_resource` | — |

The "Create environment" process creates a Kubernetes namespace with Deckhouse Development Portal (DDP) labels.
The "TTL" parameter accepts the values "1 day", "3 days", "7 days", "14 days", and "1 month" and is written to the `ddp.deckhouse.io/deletion-ttl` label; an empty value creates a permanent environment.
DDP does not delete environments by TTL on its own, since the label is intended for an external cluster cleanup mechanism.

The "Delete environment" process deletes the namespace with all deployments, waits up to 5 minutes for it to be deleted, and updates the catalog.

For an example of adding your own process, see [Custom resource ordering process](../examples/home-custom-process/).
