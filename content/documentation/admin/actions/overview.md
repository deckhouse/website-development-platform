---
title: Overview
weight: 10
---

Actions are a platform mechanism for initiating operations in external infrastructure systems and services, i.e., outside the platform. They can be used, for example:

- Create projects, variables, branches, or tags in GitLab;
- Create resources in Kubernetes;
- Create secrets in Deckhouse Stronghold or HashiCorp Vault;
- Create projects in SonarQube, DefectDojo, and other systems;
- Create topics and ACLs in Kafka.

An action can be bound to one or more resources. After that, it can be executed for any data entity related to those resources.

## Action Configuration

### General Information

When creating or editing an action, provide the following basic information:

- **Name** (required) — an arbitrary name for the action.
- **Identifier** (required) — the action identifier. Generated automatically from the name.
- **Resource** (optional) — One or more resources for which the action will be available to run.
- **Icon** (optional) — The icon displayed on the action card.
- **Owner** (optional) — The user account responsible for the configuration and operation of the action.
- **Owner Team** (optional) — The team responsible for the configuration and operation of the action.
- **Tags** (optional) — Tags for classifying and searching for actions.
- **Description** (optional) — A description in Markdown. Displayed when the action or a script containing the action is run.

### User Form

#### Parameters

In the "User Form" section, specify the parameters available for filling when the action is run. Parameter types are described in the ["Parameters"](../../user/properties/) section.

The following options are available for each parameter:

- **Editable** — Allows you to change the parameter value when the action is launched. If this option is disabled, the value cannot be changed.
- **Required** — Requires a value to be specified when the action is launched.
- **Hidden** — The parameter is not displayed in the user form when the action is launched.

For each parameter, you can specify a default value, which will be populated in the form when the action is launched.

{{< alert level="info" >}}
For non-editable or hidden parameters, it is recommended to specify a default value, as the user will not be able to change them when the action is launched.
{{< /alert >}}

The parameter description is displayed when the action is launched by clicking the "info" button. It is recommended to fill it out to make it easier to understand the purpose of the parameters and reduce the risk of errors during launch.

You can use [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax) template functions in parameter values. For example, the expression `{{ .entity.name }}` in the parameter value means that when the action is triggered, the name of the entity for which it is triggered will be substituted.

#### Parameter Conditions

Parameters can be automatically hidden or shown in the user form depending on the value of a Boolean parameter. This is configured in the "Parameter Conditions" section, where you define the rules for showing or hiding the selected parameters.

### Backend

#### Type

The action can be executed using one of two backend types:

- **Built-in** — the main action logic is executed within the platform.

> When selecting the built-in backend, you must specify the type of built-in action. Depending on the selected type, the platform automatically generates a sample request body and determines the list of credentials required to execute the action.

- **Webhook** — the main action logic is executed by an external service to which the platform sends an HTTP request.

#### Masking Action Fields

For each action, you can enable masking of fields that may contain sensitive information.

If you enable the "Mask Action Fields" option, the following will be hidden in the action execution records:

- The body field (request body);
- The response field (response generated based on the execution results);
- The values of all filled-in parameters.

#### Temporary Response

For each action, you can enable the "Temporary Response" option. This allows you to restrict access to the action's results and hide them from other users.

If this option is enabled, after the action is successfully completed, the response:

- is saved as temporary and displayed **only** to the user who initiated the action;
- is removed from the regular response field so that it is not accessible to other users.

Other users will see only the encrypted temporary response instead of the response content.

The action initiator can delete the temporary response manually.

{{< alert level="info" >}}
The temporary response is only available to the action initiator. Other users cannot view its contents or delete it.
{{< /alert >}}

#### Number of Restarts

If an action fails, the platform can automatically retry. The "Number of Restarts" setting specifies the maximum number of such retries.

#### Base Delay (sec)

The "Base Delay" parameter specifies the interval between retries. With each retries, the delay increases exponentially. For example, with a base delay of 2 seconds, retries will occur after 2, 4, 8 seconds, and so on.

#### Request Body

Each action sends an HTTP request to the built-in backend or to the backend webhook URL. The request typically includes a body that describes the operation specification.

For the built-in backend, a sample request body is available when editing the action.

#### Request Body Example

Parameter values from a custom form can be inserted into the request body using [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax) YAML templates.

Example:

```yaml
project_id: {{ .project_id }}
```

With this request body, the expression `{{ .project_id }}` will be replaced with the value of the `project_id` parameter entered by the user in the action launch form.

#### Final Request Body

The "Final Request Body" field shows what the request will look like after parameter substitution and formatting. If the final body is valid JSON, syntax highlighting will be available in the interface. If not, check the correctness of the formed request body.

#### URL

To execute a webhook action, in the **URL** field, specify the address of the external backend to which the platform will send the request.

For built-in actions, the **URL** field can contain the address of the infrastructure service being interacted with or be left blank. Valid options depend on the specific type of built-in action and are described in its documentation.

#### Disable SSL verification

This parameter determines whether the platform verifies the SSL certificate of:

- webhook backend (for webhook actions);
- infrastructure service (for built-in actions).

Enable this option if self-signed or untrusted certificates are used.

#### Method

HTTP method of requesting the webhook backend. For built-in actions, the method is determined automatically depending on the action type.

#### Request body format

For webhook actions, you can select the format for sending the request body:

- **JSON** (default) — the request body is sent in JSON format with the `Content-Type: application/json` header.
- **Form URL Encoded** — the request body is sent in the `application/x-www-form-urlencoded` format. This format only supports flat key-value structures, where all values are converted to strings. Nested objects and arrays are not supported.

{{< alert level="info" >}}
When selecting the Form URL Encoded format, the request body must contain only flat key-value pairs. For example:

```yaml
token: {{ .credentials.token }}
```

Nested structures and arrays are not supported in this format.
{{< /alert >}}

#### Advanced Logging

Webhook actions have an **advanced logging** option, which enables detailed logging of all HTTP request and response details:

- Request URL,
- HTTP method,
- HTTP headers (including tokens),
- Request body,
- Response status code,
- HTTP response headers,
- Response body.

When enabled, all this data is written to the action execution log. When disabled, logging occurs in standard mode.

{{< alert level="warning" >}}
Enabling advanced logging may result in sensitive information (tokens, passwords, etc.) being written to the logs. Use this option with caution and only for debugging purposes.
{{< /alert >}}

#### Headers

HTTP headers in the "key: value" format that will be added to the request to the webhook backend. For built-in actions, headers are generated automatically depending on the action type.

### Entity Update

#### Entity Parameter Update

After executing an action, the result (usually the infrastructure system's response) is stored in the 'response' field. If the "Update Entity Parameters" option is enabled, the platform uses the data from the 'response' field and applies update rules to write values to the entity parameters.

For example, when executing the "Create Project in GitLab" action, the `response` field will contain the specification of the created project, such as:

```json
{
"id": "1",
"...": "..."
}
```

If you need to immediately populate the `repository_id` entity parameter after creating a project in GitLab, specify the following in the update rules:

- **source:** `{{ .response.id }}`;
- **entity parameter:** `repository_id`.

#### Write to Process Storage

After executing the action, the result is written to the **response** field. The "Write to Process Storage" block is available in the "Update" section of the action configuration. It is only used when executing an action within a process: if rules for writing to storage are defined, then after the action successfully completes, the values corresponding to these rules are written to storage.

This block specifies a list of rules:

| Field | Description | Examples |
|------|----------|---------|
| **Source** | A Go template string where the context is the **response** of this action. The template is executed after the action successfully completes; the result (string) is written to storage. | `{{ .id }}` — get the id field from the response; `{{ .result.projectId }}` — get the projectId nested field from the result in the response |
| **Storage Key** | The name of the key in the process storage. The value from the source is stored under this key. | `projectId`, `deployJobId` |

If there are no rules (the list is empty), nothing is written to storage when the action in the process is executed. Data from the storage can be used in subsequent actions in the same process via the `{{ .store.<key> }}` placeholders. For more information on using storage, see the ["Process Storage"](../processes/#process-storage) section.

{{< alert level="info" >}}
Storage is written only after the action has successfully completed.
{{< /alert >}}

#### Updating User Credentials

After the action has completed, the result is written to the **response** field. If the **update user credentials** option is enabled, the action automatically updates the user's credentials based on the data in **response** according to the update rules.

For example, when performing an action that returns a new API key in the response:

```json
{
"apiKey": "new-api-key-12345",
"...": "..."
}
```

To automatically update user credentials, specify the following in the update rules:

* **Source**: `{{ .response.apiKey }}` — a template for retrieving the value from the action response.
* **Credential Type**: Select the type of credentials to be updated.

##### Selecting a user to update credentials

By default, credentials are updated for the user who launched the action (the initiator). To update the credentials of a different user, enable the **Update credentials for a specific user** option and select the desired user.

{{< alert level="info" >}}
Credentials are updated only after the action has successfully completed. If the action fails, credentials are not updated.
{{< /alert >}}

#### Creating Entity Relationships

If the "Create Entity Relationships" option is enabled, the platform will automatically create new relationships for the selected entity based on the specified rules. The set of rules is defined separately for each resource.

A relationship rule specifies the **parent** and **child** resources. Entities are searched for by a single identifier: either the parent resource or the child resource.
Within a single rule, specify **only one** identifier. If both parent and child identifiers are specified, the search is performed by the **parent** resource.

### Security

#### Confirmation before launch

You can enable mandatory confirmation for an action and specify the number of approvers. In this case, the action will not launch until the specified number of confirmations are received from the specified users.

The current number of confirmations and the list of users awaiting confirmation are displayed in the action launch table for the corresponding entity.

If confirmation is required from a specific user, they will be notified in the platform interface.

#### Account to launch the action

By default, access to external infrastructure services is performed using the credentials of the user who launched the action. If necessary, you can explicitly specify that the action should be executed under a specific account.

#### Credentials

For built-in actions, the platform predefines a set of required credentials. Their IDs are loaded when selecting the built-in backend. For each ID, you must select the credential type to use.

For webhook actions, credentials can be referenced in the request body using the `{{ .credentials.<credential type ID> }}` construct.

## Action Runs

### Action Run Records

Each action run creates a record containing the initiator, execution status, and a detailed log. Run records for each entity are available in the entity card on the "Action Runs" tab. If an action requires confirmation, it is executed in the same tab.

A run record can be deleted or the action can be restarted. A restart creates a new record.

### Run Logging

Each action run creates a record in the database containing the full execution log.

The `actions.logging.enabled` parameter in the DDP configuration file controls whether startup logs are output to stdout: if `true`, logs are output; if `false`, they are not.

{{< alert level="info" >}}
Entries with the full startup log are created in the database regardless of the `actions.logging.enabled` value.
{{< /alert >}}

### Action Statuses

For each action execution, a status entry is created. Possible statuses:

- **Created** — the entry has been created, but the action has not yet been executed.
- **Unapproved** — the action is awaiting approval.
- **Running** — the action is running.
- **Failed** — the action completed with an error.
- **Update failed** — the action completed, but updating the entity parameters failed.
- **Success** — the action completed successfully.
- **Retrying** — the action completed with an error and is being attempted again.