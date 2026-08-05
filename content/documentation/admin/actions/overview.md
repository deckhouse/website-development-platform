---
title: Overview
description: Action configuration in Deckhouse Development Platform (DDP) — requests, form, entity updates, security, and authorization.
weight: 10
---

Actions are a platform mechanism for running operations in external infrastructure systems and services. With actions, for example, you can:

- create projects, variables, branches, tags, releases, and merge requests in [GitLab](../gitlab/);
- create resources in [Kubernetes](../kubernetes/) and retrieve them;
- create secrets in [Deckhouse Stronghold and HashiCorp Vault](../vault/), as well as clients in [Keycloak](../keycloak/);
- create projects in [SonarQube](../sonarqube/), products and engagements in [DefectDojo](../defectdojo/), projects in [CodeScoring](../codescoring/), and repositories, roles, users, and privileges in [Nexus Repository](../nexus/);
- create topics and ACLs in [Apache Kafka](../kafka/);
- perform helper actions for debugging and controlling process execution (for example, [Debug](../debug/) and [Wait](../wait/)).

An action can be linked to one or more resources. Once linked, it can be run for any entity of those resources.

## Action configuration

### Basic information

When creating or editing an action, specify the basic information:

| Field             | Required | Description                                                                                |
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

#### Action type

An action can be of the following types:

- "Built-in" — the action's execution logic is described inside the platform. For built-in actions, you must select one of the preconfigured backends.
- "Webhook" — the action's execution logic is fully configured by the user.

#### Retry parameters

If an action fails, the platform can automatically retry it.

##### Number of retries

How many times to retry execution after a failure. 0 means a single attempt with no retries.

##### Base delay (sec.)

The delay in seconds before the first retry; the pause doubles before each subsequent attempt (exponential backoff).

#### Request body format

For webhook actions, you can choose the format used to send the request body:

- JSON (default) — the request body is sent in JSON format with the `Content-Type: application/json` header.
- "Form URL Encoded" — the request body is sent in `application/x-www-form-urlencoded` format.

{{< alert level="info" >}}
When "Form URL Encoded" is selected, the request body must contain only flat key-value pairs. Nested structures and arrays are not supported in this format.
{{< /alert >}}

Example:

```yaml
token: {{ .credentials.token }}
```

#### Extended logging

For webhook actions, you can enable an extended logging option that provides detailed logging of all HTTP request and response details:

- request URL;
- HTTP method;
- HTTP headers (including tokens);
- request body;
- response status code;
- response HTTP headers;
- response body.

When this option is enabled, all this data is written to the action run log. When it is disabled, logging works in standard mode.

{{< alert level="warning" >}}
Enabling extended logging may result in sensitive information (tokens, passwords, etc.) being written to the logs. Use this option with caution and only when needed for debugging.
{{< /alert >}}

#### Request body

Each action sends an HTTP request to a built-in backend or to a webhook backend URL. The request usually includes a request body (`body`) that describes what exactly will be sent. The request body is defined in YAML.

Parameter values from the user form can be substituted into the request body using [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax) templates.

Example:

```yaml
project_id: {{ .property.project_id }}
```

With this request body, the expression `{{ .property.project_id }}` is replaced with the value of the `project_id` parameter that the user entered in the action's launch form.

For built-in action types, the "Request body" parameter is part of the action's configuration, and a sample of this configuration is available.

### User form

#### Parameters

The "User form" section specifies the parameters that the user can fill in when launching the action. Parameter types are described in the [Parameters](../../user/properties/) section.

The following options are available for each parameter:

- "Editable" — allows the parameter value to be changed when launching the action. If this option is disabled, the value cannot be changed.
- "Required" — requires a value to be specified when launching the action.
- "Hidden" — the parameter is not displayed in the user form when launching the action.

Each parameter can have a default value that is pre-filled in the form when the action is launched.

{{< alert level="info" >}}
For non-editable or hidden parameters, it is recommended to set a default value, since the user will not be able to change them when launching the action.
{{< /alert >}}

The parameter description is shown when launching the action by clicking the "info" icon button. It is recommended to fill it in to make it easier to understand the purpose of the parameters and reduce the risk of errors when launching the action.

Parameter values can use [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax) template functions. For example, the expression `{{ .entity.name }}` in a parameter value means that when the action is launched, the name of the entity it is running for will be substituted.

#### Parameter conditions

Parameters can be automatically hidden or shown in the user form depending on the value of a `Boolean`-type parameter. This is configured in the "Parameter conditions" section, where you define rules for showing or hiding selected parameters.

### Update

#### Templating contexts

All updates are performed only after the action completes successfully. The following templating contexts are available in Go templates:

| Context     | Description                                                               | Example                              |
| ----------- | --------------------------------------------------------------------------- | ------------------------------------- |
| `.property` | Values from the user form                                                   | `{{ .property.repo }}`                |
| `.response` | Parsed action response (JSON)                                               | `{{ .response.id }}`                  |
| `.entity`   | Parameters of the catalog entity for which the action is run                | `{{ .entity.slug }}`                  |
| `.workflow` | Workflow parameters                                                         | `{{ .workflow.branch }}`              |
| `.process`  | Process parameters                                                          | `{{ .process.releaseId }}`            |
| `.global`   | Global variables (variable set identifier, then key)                        | `{{ .global.myGroup.apiUrl }}`        |
| `.team`     | Variables of the team selected at launch                                    | `{{ .team.token }}`                   |
| `.store`    | Variables from the process store                                            | `{{ .store.myKey }}`                  |

#### Entity parameter update

If the "Update entity parameters" option is enabled, the platform applies the update rules and writes the values to the entity's parameters.

| Field               | Description                                                              |
| ------------------- | ------------------------------------------------------------------------- |
| Source              | Go template for the value                                                 |
| Entity parameter    | Identifier of the entity parameter in the catalog                         |

For example, when the "Create GitLab project" action is run, the response (`response`) will contain the specification of the created project:

```json
{
  "id": "1",
  "...": "..."
}
```

If you need to immediately populate the entity's `repository_id` parameter after creating a project in GitLab, specify the following update rule:

1. Source: `{{ .response.id }}`.
1. Entity parameter: `repository_id`.

#### Entity creation

If the "Create entities" option is enabled, the platform automatically creates new entities in the selected resources according to the specified rules. Rules are defined separately for each resource.

| Field                                | Description                                                                                  |
| ------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Owner of created entities             | Assigned as the owner of all entities the action creates in the catalog                       |
| Owner team of created entities        | Assigned as the owner team of all entities the action creates in the catalog                   |
| Resource                              | Catalog resource in which the entity will be created                                          |
| Source (identifier)                   | Go template for the identifier of the entity being created                                     |
| Source (name)                         | Go template for the name of the entity being created                                          |
| Additional rules                      | Mapping of a Go template value to an entity parameter                                          |

The identifier and name are required for each rule group. If a resource already has an entity with the same identifier, creation is skipped.

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

If needed, add additional rules to populate entity parameters, for example map `{{ .response.path }}` to the `path` parameter.

#### Entity relation creation

If the "Create entity relations" option is enabled, the platform automatically creates new relations for the selected entity according to the specified rules. The set of rules is defined separately for each resource.

| Field                              | Description                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Resource                             | Either of the two resources involved in the required resource relation: the list of relations in which the selected resource is specified as the parent or the child is loaded based on the selected resource. You can choose the resource of the entity for which the action is running, or the resource on the other side of the relation — the list will contain the same relation if both resources are part of it |
| Relation                             | The catalog resource relation: it defines the parent and child resources and entity roles when creating the relation. The selected relation determines which side the launch entity is on and which of the fields below specifies the identifier of the second entity from the action's response                                        |
| Parent entity identifier             | Go template for the identifier of the parent entity in the catalog                                                                                                                                                                                                                                                                          |
| Child entity identifier              | Go template for the identifier of the child entity in the catalog                                                                                                                                                                                                                                                                           |

#### User credentials update

If the user credentials update option is enabled, the action automatically updates the user's credentials according to the update rules.

| Field               | Description                          |
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

to update the credentials, specify the following update rules:

1. Source: `{{ .response.apiKey }}`.
1. Credentials type: select the type of credentials to be updated.

##### Selecting a user for credentials update

By default, credentials are updated for the user who launched the action (the initiator).

To update the credentials of a different user, enable the "Update credentials of a specific user" option and select the desired user.

#### Process store update

Rules from the "Process store update" section only apply when the action is run as part of a process: after the action completes successfully, values are written to the run's store according to the specified rules, in list order.

| Field      | Description                                                                                                       |
| ----------- | -------------------------------------------------------------------------------------------------------------------- |
| Condition  | An optional Go template; if empty, the rule always applies. Rendering result: `true`, `false`, `1`, or `0`          |
| Operation  | Write string, Write JSON, Append string, Append JSON, Merge JSON (shallow), Delete                                  |
| Target     | Dot-path in the store without a leading dot, e.g. `deploy.job_id` or `notification.items`                            |
| Source     | Go template for the value                                                                                             |

Other actions and process elements read this data via `{{ .store.<path> }}`. Inside a loop, the `{{ .store._loop.* }}` context is available in templates.

If there are no rules (the list is empty), nothing is written to the store when the action runs as part of a process.

For a detailed description of operations, rule examples, iterating over a collection, and viewing the store, see [Process store](../processes/store/).

### Security

#### Execution conditions

The action will only run if the entity's status matches one of the selected values in the status field. If no restrictions are set, this condition does not apply.

#### Masking action fields

For each action, you can enable masking of fields that may contain sensitive information.

If this option is enabled, the following will be hidden in the action's run records:

- values of all filled-in parameters;
- request body (`body`);
- response (`response`) generated as a result of execution.

#### Temporary response

For each action, you can enable the "Temporary response" option. It restricts access to the action's execution result and hides it from other users.

If this option is enabled, after the action completes successfully, the response:

- is stored as temporary and displayed only to the user who launched the action;
- is removed from the regular response field so that it is not accessible to other users.

Other users will see only an encrypted placeholder instead of the response content.

The action's initiator can delete the temporary response manually.

{{< alert level="info" >}}
The temporary response is available only to the action's initiator. Other users cannot view its contents and cannot delete it.
{{< /alert >}}

#### Approval

You can enable mandatory approval for an action and specify the number of approvers required. In this case, the action will not run until the specified number of approvals has been received from the designated users.

The current number of approvals and the list of users whose approval is expected are displayed in the action run table for the corresponding entity.

If approval is required from a specific user, they will receive a notification in the web interface.

#### Approval notifications

When approval is enabled, you can enable notifications and specify how they are sent. If sending to a URL is selected, specify the webhook address, optionally disable the server's TLS certificate verification, and add additional HTTP request headers.

### Authorization

#### Use external service

Selection of the external services for which the action can be run. If multiple external services are added, a specific one can be selected directly when launching the action.

#### URL and HTTP headers

The address to which the request will be sent, and the HTTP headers.

Headers support Go templates in the form `{{ .credentials.<credentials type identifier> }}` for substituting user credentials.

#### Select an account to run as

By default, external infrastructure services are accessed using the credentials of the user who launched the action. If needed, you can explicitly specify that the action should run on behalf of a specific account.

For actions launched as automation events, specifying the account to run as is mandatory.

#### Credentials

For built-in actions, the platform predefines the set of required credentials. Their identifiers are loaded when the built-in backend is selected. For each identifier, you must select the type of credentials to be used.

For webhook actions, credentials can be accessed in the request body using the `{{ .credentials.<credentials type identifier> }}` construct.

#### HTTP method

For webhook actions, the HTTP method of the outgoing request is specified.

## Action runs

### Action run records

Each time an action is run, a record is created containing the initiator, execution status, and a detailed log. Run records for each entity are available on the entity's card, on the "Action runs" tab. If the action requires approval, it is handled on the same tab.

A run record can be deleted, or the action can be re-run. Re-running creates a new record.

### Run logging

For each action run, a record containing the full execution log is created in the database.

The `actions.logging.enabled` parameter in the DDP configuration file controls whether run logs are output to `stdout`: when set to `true`, logs are output; when set to `false`, they are not.

{{< alert level="info" >}}
Records with the full run log are created in the database regardless of the value of `actions.logging.enabled`.
{{< /alert >}}

### Action run statuses

For each action run, a record with a status is created. Possible statuses:

- `Created` — the record has been created, but the action has not been run yet.
- `Unapproved` — the action is awaiting approval.
- `Running` — the action is running.
- `Failed` — the action finished with an error.
- `Update failed` — the action completed, but updating the entity's parameters failed.
- `Success` — the action completed successfully.
- `Retrying` — the action finished with an error and is being retried.
