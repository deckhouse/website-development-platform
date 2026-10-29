---
title: Overview
description: Action configuration in Deckhouse Development Portal (DDP) — requests, form, entity updates, security, and authorization.
weight: 10
---

Actions are a Deckhouse Development Portal mechanism for running operations in external infrastructure systems and services.
For example, actions can:

- Create projects, variables, branches, tags, releases, and merge requests in [GitLab](../gitlab/).
- Create resources in [Kubernetes](../kubernetes/) and retrieve them.
- Create secrets in [Deckhouse Stronghold and HashiCorp Vault](../vault/), as well as clients in [Keycloak](../keycloak/).
- Create projects in [SonarQube](../sonarqube/), products and engagements in [DefectDojo](../defectdojo/), projects in [CodeScoring](../codescoring/), and repositories, roles, users, and privileges in [Nexus Repository](../nexus/).
- Create topics and ACLs in [Apache Kafka](../kafka/).
- Perform helper actions for debugging and controlling process execution (for example, [Debug](../debug/) and [Wait](../wait/)).

An action can be linked to one or more resources. Once linked, it can be run for any entity of those resources.

## Action configuration

An action is configured on several tabs described below.

### Basic information

When creating or editing an action, specify the basic information:

| Name              | Required | Description                                                                                |
| ----------------- | -------- | -------------------------------------------------------------------------------------------- |
| Name               | Yes      | Arbitrary name of the action                                                                |
| Identifier         | Yes      | Action identifier. Generated automatically from the name                                    |
| Resource           | No       | One or more resources for which the action will be available to run                         |
| Icon               | No       | Icon displayed on the action card                                                            |
| Owner              | No       | User account responsible for the action's configuration and operation                       |
| Owner team         | No       | Team responsible for the action's configuration and operation                                |
| Tags               | No       | Tags used to classify and search for actions                                                 |
| Description        | No       | Description in Markdown. Displayed when running the action or a process that contains it     |

### Request configuration

The request configuration defines how the action is executed and what it sends.

#### Action type

An action can be built-in or a webhook.
For a "Built-in (BuiltIn)" action, the execution logic is defined within the portal.
For built-in actions, you must select one of the preconfigured backends.
For a "Webhook" action, the execution logic is fully configured by the user.

#### Restart parameters

If an action fails, the portal can automatically retry it. Configure retries in the "Restart parameters" section:

| Name           | Description                                                                                                       |
| -------------- | ----------------------------------------------------------------------------------------------------------------- |
| Max retries    | How many times to retry execution after a failure. `0` means a single attempt with no retries                     |
| Base delay (s) | Delay in seconds before the first retry. The pause doubles before each subsequent attempt (exponential backoff)  |

#### Request body format

For webhook actions, you can choose the format used to send the request body.
By default, the request body is sent in JSON format with the `Content-Type: application/json` header.
With "Form URL Encoded", the request body is sent in `application/x-www-form-urlencoded` format.

{{< alert level="info" >}}
When "Form URL Encoded" is selected, the request body must contain only flat key-value pairs.
Nested structures and arrays are not supported in this format.
{{< /alert >}}

If the request body is empty, the webhook sends the request without a body and without the `Content-Type` header.
Use this for `GET` and `DELETE` requests.

If the external system returns an error response code, the response body is saved in the action response: JSON as an object, anything else in the `raw_body` field.
The action error text has the form `unexpected status code: <CODE>, response: <RESPONSE_BODY>`; the response body is truncated to 8192 bytes.

Example:

```yaml
token: {{ .credentials.token }}
```

#### Verbose logging

For webhook actions, you can enable the "Verbose logging" option that logs all HTTP request and response details:

- Request URL
- HTTP method
- HTTP headers (including tokens)
- Request body
- Response status code
- Response HTTP headers
- Response body

When this option is enabled, all this data is written to the action run log.
When it is disabled, logging works in standard mode.

{{< alert level="warning" >}}
Enabling verbose logging may result in sensitive information, such as tokens and passwords, being written to the logs.
Use this option with caution and only when needed for debugging.
{{< /alert >}}

#### Request body

Each action sends an HTTP request to a built-in backend or to a webhook backend URL.
The request usually includes a request body (`body`) that describes what exactly will be sent.
The request body is defined in YAML.

Parameter values from the user form can be substituted into the request body using [Go template](https://pkg.go.dev/text/template) templates.

Example:

```yaml
project_id: {{ .property.project_id }}
```

With this request body, the expression `{{ .property.project_id }}` is replaced with the value of the `project_id` parameter that the user entered in the action's launch form.

For built-in actions, the "Request body" parameter is part of the action configuration, and a configuration example is available.

### User form

The user form defines the parameters that the user fills in when running the action.

#### Parameters

The "User form" section specifies the parameters that the user can fill in when launching the action.
Property types are described in the [Properties](../../user/properties/) section.

The following options are available for each parameter:

- "Editable": Allows the parameter value to be changed when launching the action.
  If this option is disabled, the value cannot be changed.
- "Required": Requires a value to be specified when launching the action.
- "Hidden": The parameter is not displayed in the user form when launching the action.

Each parameter can have a default value that is pre-filled in the form when the action is launched.

{{< alert level="info" >}}
Set a default value for non-editable and hidden parameters: the user cannot change them when launching the action.
{{< /alert >}}

The parameter description is shown when launching the action by clicking the "info" icon.
Providing a description makes the parameter's purpose easier to understand and reduces the risk of errors when launching the action.

Parameter values can use [Go template](https://pkg.go.dev/text/template) template functions.
For example, the expression `{{ .entity.name }}` in a parameter value means that when the action is launched, the name of the entity it is running for will be substituted.

#### Property conditions

Parameters can be automatically hidden or shown in the user form depending on the value of a `Boolean`-type parameter.
This is configured in the "Property conditions" section, where you define rules for showing or hiding selected parameters.

### Update

After a successful run, the action can update entities, relations, credentials, and the process store.

#### Templating contexts

All updates are performed only after the action completes successfully.
The following templating contexts are available in Go templates:

| Context     | Description                                                               | Example                              |
| ----------- | --------------------------------------------------------------------------- | ------------------------------------- |
| `.property` | Values from the user form                                                   | `{{ .property.repo }}`                |
| `.response` | Parsed action response (JSON)                                               | `{{ .response.id }}`                  |
| `.entity`   | Properties of the catalog entity for which the action is run                | `{{ .entity.slug }}`                  |
| `.workflow` | Workflow parameters                                                         | `{{ .workflow.branch }}`              |
| `.process`  | Process parameters                                                          | `{{ .process.releaseId }}`            |
| `.global`   | Global variables (variable set identifier, then key)                        | `{{ .global.myGroup.apiUrl }}`        |
| `.team`     | Variables of the team selected at launch                                    | `{{ .team.token }}`                   |
| `.store`    | Variables from the process store                                            | `{{ .store.myKey }}`                  |
| `.context`  | [Process launch context](../processes/overview/#launch-context), for example data of the environment selected on the Home page | `{{ .context.environment.slug }}` |

#### Entity property update

To enable property update, turn on the switch in the "Entity property updates" section header.
The portal then applies the update rules and writes the values to the entity's properties.

| Name                | Description                                                              |
| ------------------- | ------------------------------------------------------------------------- |
| Source              | Go template for the value                                                 |
| Entity property     | Identifier of the entity property in the catalog                          |

For example, when the "Create GitLab project" action is run, the response (`response`) will contain the specification of the created project:

```json
{
  "id": "1",
  "...": "..."
}
```

If you need to immediately populate the entity's `repository_id` property after creating a project in GitLab, specify the following update rule:

1. Source: `{{ .response.id }}`.
1. Entity property: `repository_id`.

#### Entity creation

To enable entity creation, turn on the switch in the "Creating entities" section header.
The portal then automatically creates new entities in the selected resources according to the specified rules.
Rules are defined separately for each resource.

| Name                                 | Description                                                                                  |
| ------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Owner is the run initiator            | The user who launched the action becomes the owner of created entities. If the initiator is unknown (a run by an automation or a webhook), the owner from the "Owner of created entities" field is used |
| Owner of created entities             | Assigned as the owner of all entities the action creates in the catalog                       |
| Owning team of created entities       | Assigned as the owner team of all entities the action creates in the catalog                   |
| Resource                              | Catalog resource in which the entity will be created                                          |
| Source (identifier)                   | Go template for the identifier of the entity being created                                     |
| Source (name)                         | Go template for the name of the entity being created                                          |
| Additional rules                      | Mapping of a Go template value to an entity property                                           |

The identifier and name are required for each rule group.
If a resource already has an entity with the same identifier, creation is skipped.

For example, when the "Create GitLab project" action is run, the response may contain:

```json
{
  "id": "42",
  "name": "my-project",
  "...": "..."
}
```

To create an entity in the "GitLab projects" resource after the action completes successfully, specify the following rules:

1. Resource: "GitLab projects".
1. Source (identifier): `{{ .response.id }}`.
1. Source (name): `{{ .response.name }}`.

If needed, add additional rules to populate entity properties, for example map `{{ .response.path }}` to the `path` property.

#### Entity deletion

To enable entity deletion, turn on the switch in the "Deleting entities" section header.
The portal then automatically deletes entities in the selected resources according to the specified rules.
Rules are defined separately for each resource.

| Name     | Description                                                                 |
| -------- | --------------------------------------------------------------------------- |
| Resource | Catalog resource that contains the entity to delete                         |
| Source   | Go template for the entity identifier (for example, `{{ .response.id }}`)   |

If the resource has no entity with the resulting identifier, deletion is skipped.

For example, when the "Delete GitLab project" action is run, the response may contain:

```json
{
  "id": "42",
  "...": "..."
}
```

To delete the entity in the "GitLab projects" resource after the action completes successfully, specify the following rules:

1. Resource: "GitLab projects".
1. Source: `{{ .response.id }}`.

#### Entity relation creation

To enable relation creation, turn on the switch in the "Creating relations for the entity" section header.
The portal then automatically creates new relations for the selected entity according to the specified rules.
The set of rules is defined separately for each resource.

| Name                               | Description                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Resource                             | Either of the two resources involved in the required resource relation: the list of relations in which the selected resource is specified as the parent or the child is loaded based on the selected resource. You can choose the resource of the entity for which the action is running, or the resource on the other side of the relation — the list will contain the same relation if both resources are part of it |
| Relation                             | The catalog resource relation: it defines the parent and child resources and entity roles when creating the relation. The selected relation determines which side the launch entity is on and which of the fields below specifies the identifier of the second entity from the action's response                                        |
| Parent entity identifier             | Go template for the identifier of the parent entity in the catalog                                                                                                                                                                                                                                                                          |
| Child entity identifier              | Go template for the identifier of the child entity in the catalog                                                                                                                                                                                                                                                                           |

#### Entity relation deletion

To enable relation deletion, turn on the switch in the "Deleting relations for the entity" section header.
The portal then automatically deletes relations for the selected entity according to the specified rules.
The set of rules is defined separately for each resource.

| Name                      | Description                                                                                                                                                                                                                                                                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Resource                  | Either of the two resources involved in the required resource relation: the list of relations in which the selected resource is specified as the parent or the child is loaded based on the selected resource. You can choose the resource of the entity for which the action is running, or the resource on the other side of the relation — the list will contain the same relation if both resources are part of it |
| Relation                  | The catalog resource relation: it defines the parent and child resources and entity roles when deleting the relation. The selected relation determines which side the launch entity is on and which of the fields below specifies the identifier of the second entity from the action's response                                         |
| Parent entity identifier  | Go template for the identifier of the parent entity in the catalog                                                                                                                                                                                                                                                                          |
| Child entity identifier   | Go template for the identifier of the child entity in the catalog                                                                                                                                                                                                                                                                           |

If no matching relation exists, deletion is skipped.

#### User credentials update

To enable user credentials update, turn on the switch in the "Updating user credentials" section header.
The action then automatically updates the user's credentials according to the specified rules.

| Name                | Description                          |
| ------------------- | --------------------------------------- |
| Source              | Go template for the value               |
| Credentials type    | Type of credentials to be updated       |

For example, when running an action that returns a new API key in its response:

```json
{
  "apiKey": "new-api-key-12345",
  "...": "..."
}
```

To update the credentials, specify the following rules:

1. Source: `{{ .response.apiKey }}`.
1. Credentials type: select the type of credentials to be updated.

By default, credentials are updated for the user who launched the action (the initiator).
To update the credentials of a different user, enable the "Update credentials for specific user" option and select the user in the "User for credentials update" field.

#### Process store update

Rules from the "Updating process store" section only apply when the action is run as part of a process: after the action completes successfully, values are written to the run's store according to the specified rules, in list order.

| Name       | Description                                                                                                       |
| ----------- | -------------------------------------------------------------------------------------------------------------------- |
| Condition  | An optional Go template; if empty, the rule always applies. Rendering result: `true`, `false`, `1`, or `0`          |
| Operation  | Write string, Write JSON, Append string, Append JSON, Merge JSON (shallow), Delete                                  |
| Target     | Dot-path in the store without a leading dot, for example, `deploy.job_id` or `notification.items`                            |
| Source     | Go template for the value                                                                                             |

Other actions and process elements read this data via `{{ .store.<PATH> }}`.
Inside a loop, the `{{ .store._loop.* }}` context is available in templates.

If there are no rules (the list is empty), nothing is written to the store when the action runs as part of a process.

For a detailed description of operations, rule examples, iterating over a collection, and viewing the store, see [Process store](../processes/store/).

#### Datasource synchronization

Datasource synchronization after an action brings the object that the action created or deleted in the external system into the catalog without waiting for the scheduled synchronization.

To enable synchronization, turn on the switch in the "Datasource synchronization" section header and select datasources in the "Datasources" field.
Select the datasources that load the object created or deleted by the action into the catalog.
If the switch is on, select at least one datasource.

Synchronization starts after the action completes successfully and the update rules are applied.
Synchronization runs in the background: the action does not wait for it to finish and does not fail if synchronization fails.
Deleted datasources are skipped.

### Security

Security settings restrict when and by whom the action can be run.

#### Execution conditions

The action will only run if the entity's status matches one of the selected values in the status field.
If no restrictions are set, this condition does not apply.

#### Masking action fields

For each action, you can enable masking of fields that may contain sensitive information.

If this option is enabled, the following will be hidden in the action's run records:

- Values of all filled-in parameters.
- Request body (`body`).
- Response (`response`) generated as a result of execution.

#### Temporary response

For each action, you can enable the "Temporary response" option.
It restricts access to the action's execution result and hides it from other users.

If this option is enabled, after the action completes successfully, the response is stored as temporary and displayed only to the user who launched the action.
It is removed from the regular response field so that it is not accessible to other users.

Other users will see only an encrypted placeholder instead of the response content.

The action's initiator can delete the temporary response manually.

{{< alert level="info" >}}
The temporary response is available only to the action's initiator.
Other users cannot view its contents and cannot delete it.
{{< /alert >}}

#### Approval

You can enable mandatory approval for an action and specify the number of approvers required.
In this case, the action will not run until the specified number of approvals has been received from the designated users.

The current number of approvals and the list of users whose approval is expected are displayed in the action run table for the corresponding entity.

If approval is required from a specific user, they will receive a notification in the web interface.

#### Approval notifications

When approval is enabled, you can enable notifications and specify how they are sent.
If sending to a URL is selected, specify the webhook address, optionally disable the server's TLS certificate verification, and add additional HTTP request headers.

### Authorization

Authorization settings define which external services and credentials the action uses.

#### Use external service

Select the external services for which the action can be run.
If multiple external services are added, a specific service can be selected when launching the action.

#### URL and HTTP headers

Specify the address to which the request will be sent and the HTTP headers.

Headers support Go templates in the form `{{ .credentials.<CREDENTIALS_TYPE_ID> }}` for substituting user credentials.

#### Select account for execution

By default, external infrastructure services are accessed using the credentials of the user who launched the action.
To run the action on behalf of a specific account, enable the "Select account for execution" option and select the account in the "Account for execution" field.

For actions launched as automation events, specifying the account to run as is mandatory.

#### Credentials

For built-in actions, the portal predefines the set of required credentials.
Their identifiers are loaded when the built-in backend is selected.
For each identifier, you must select the type of credentials to be used.

For webhook actions, credentials can be accessed in the request body using the `{{ .credentials.<CREDENTIALS_TYPE_ID> }}` construct.

#### HTTP method

For webhook actions, the HTTP method of the outgoing request is specified.

## Action runs

The portal records each action run and its log.

### Action run records

Each time an action is run, a record is created containing the initiator, execution status, and a detailed log.
Run records for each entity are available on the entity's card, on the "Actions" tab in the "Execution history" table.
If the action requires approval, it is handled on the same tab.

A run record can be deleted, or the action can be re-run. Re-running creates a new record.

### Run logging

For each action run, a record containing the full execution log is created in the database.

The `actions.logging.enabled` module parameter controls whether run logs are output to `stdout`: when set to `true`, logs are output; when set to `false` (default), they are not.
Set the parameter in the `settings` section of ModuleConfig `development-platform` or in the Deckhouse Platform web interface.

{{< alert level="info" >}}
Records with the full run log are created in the database regardless of the value of `actions.logging.enabled`.
{{< /alert >}}

### Action run statuses

For each action run, a record with a status is created. Possible statuses:

- `Created`: The record has been created, but the action has not been run yet.
- `Unapproved`: The action is awaiting approval.
- `Running`: The action is running.
- `Failed`: The action finished with an error.
- `Update failed`: The action completed, but updating the entity's properties failed.
- `Success`: The action completed successfully.
- `Retrying`: The action finished with an error and is being retried.
- `Warning`: The action completed, but part of the operations failed.
  For example, the CreateGitlabProjectVariables action could not create some of the variables.
- `Declined`: The action was not run because the entity status does not match the [execution conditions](#execution-conditions).
- `Skipped`: The action was not run because it was disabled in the process launch configuration.
- `Canceled`: The run was canceled by a user or because the process was stopped.
