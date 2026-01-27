---
title: Overview
weight: 10
---

Actions are a platform mechanism for running operations in external infrastructure systems and services, that is, outside the platform. For example, you can use actions to:

- create projects, variables, branches, or tags in GitLab
- create resources in Kubernetes
- create secrets in Deckhouse Stronghold or HashiCorp Vault
- create projects in SonarQube, DefectDojo, and other systems
- create topics and ACLs in Kafka

An action can be associated with one or more resources. After that, it can be run for any data entity of those resources.

## Action configuration

### Basic information

When creating or editing an action, specify the following:

- **Name** (required): An arbitrary name for the action.
- **Identifier** (required): The action identifier. It is generated automatically from the name.
- **Resource** (optional): One or more resources for which this action will be available.
- **Icon** (optional): An icon displayed in the action card.
- **Owner** (optional): The user account responsible for the action configuration and reliability.
- **Owner team** (optional): The team responsible for the action configuration and reliability.
- **Tags** (optional): Tags used to classify and search for actions.
- **Description** (optional): A Markdown description shown when running the action or a scenario that includes it.

### User form

#### Parameters

In the **User form** section, define the parameters that users can fill in when running the action. Parameter types are described in the **Parameters** section.

Each parameter supports the following options:

- **Editable**: Allows changing the parameter value when running the action. If disabled, the value cannot be changed.
- **Required**: Requires a value to be provided when running the action.
- **Hidden**: Hides the parameter from the user form when running the action.

You can also set a default value for each parameter. The platform will prefill the user form with it.

{{< alert level="info" >}}
For non-editable or hidden parameters, it is recommended to always set a default value, because users cannot change them when running the action.
{{< /alert >}}

A parameter description is available when running the action via the **info** icon. Filling in descriptions is recommended: it makes the purpose of parameters clearer and reduces the risk of mistakes.

Parameter values can use [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax). For example, using `{{ .entity.name }}` as a parameter value means the platform will substitute the name of the entity the action is being run for.

#### Parameter conditions

Parameters can be automatically shown or hidden in the user form depending on the value of a `Boolean` parameter. Configure this behavior in the **Parameter conditions** section, where you define show/hide rules for selected parameters.

### Backend

#### Type

An action can use one of two backend types:

- **Built-in (BuiltIn)**: The main action logic runs inside the platform.

  > When selecting a built-in backend, you must choose a specific built-in action type. Depending on the selected type, the platform automatically generates an example request body and determines which credentials are required to run the action.

- **Webhook (Webhook)**: The main action logic runs in an external service. The platform sends an HTTP request to that service.

#### Field masking

For each action, you can enable masking for fields that may contain sensitive information.

If **Mask action fields** is enabled, the following will be hidden in action run records:

- the `body` field (request body);
- the `response` field (response produced after the action completes);
- values of all filled-in parameters.

#### Temporary response

For each action, you can enable the **Temporary response** option. It restricts access to the action result and hides it from other users.

When the option is enabled and the action completes successfully:

- the response is stored as a temporary response and is visible **only** to the user who started the action;
- the regular response is cleared so it is not available to other users.

Other users will see only an encrypted temporary response instead of the actual content.

The action initiator can delete the temporary response manually.

{{< alert level="info" >}}
A temporary response is available only to the action initiator. Other users cannot view its contents and cannot delete it.
{{< /alert >}}

#### Retry count

If an action fails, the platform can retry it automatically. The **retry count** parameter defines the maximum number of automatic retries.

#### Base delay (sec)

The **base delay** parameter defines the delay between retries. The delay increases on each retry (exponentially). For example, with a base delay of 2 seconds, retries will occur after 2, 4, 8 seconds, and so on.

#### Request body

Each action sends an HTTP request either to the built-in backend or to the webhook backend URL. In most cases, the request includes a body describing the operation specification.

For built-in backends, an example request body is available while editing the action.

#### Request body example

You can inject parameter values from the user form into the request body using [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax) with YAML syntax.

Example:

```yaml
project_id: {{ .project_id }}
```

With this request body, the `{{ .project_id }}` expression will be replaced with the `project_id` parameter value entered by the user in the action run form.

#### Final request body

The **Final request body** field shows what the request will look like after parameter substitution and formatting. If the final body is valid JSON, the UI will display syntax highlighting. If highlighting is not available, verify that the generated request body is correct.

#### URL

To run a webhook action, specify the external backend address in the **URL** field. The platform will send the request to this address.

For built-in actions, the **URL** field can either contain the address of the infrastructure service the action interacts with or be left empty. The allowed options depend on the specific built-in action type and are described in its documentation.

#### Disable SSL verification

This option defines whether the platform verifies the SSL certificate of:

- the webhook backend (for webhook actions);
- the infrastructure service (for built-in actions).

Enable this option if you use self-signed or otherwise untrusted certificates.

#### Method

The HTTP method used for requests to the webhook backend. For built-in actions, the method is determined automatically based on the action type.

#### Headers

HTTP headers in the `key: value` format that will be added to the request to the webhook backend. For built-in actions, headers are generated automatically depending on the action type.

### Entity updates

#### Updating entity parameters

After an action runs, the result (typically the response from the infrastructure system) is stored in the `response` field. If **Update entity parameters** is enabled, the platform uses the data from `response` and applies the update rules to write values into the entity parameters.

For example, when running the **Create GitLab project** action, the `response` field will contain the specification of the created project, for example:

```json
{
  "id": "1",
  "...": "..."
}
```

If you need to populate the `repository_id` entity parameter right after creating a GitLab project, specify the following in the update rules:

- **Source:** `{{ .response.id }}`
- **Entity parameter:** `repository_id`

#### Creating entity relationships

If **Create entity relationships** is enabled, the platform will automatically create new relationships for the selected entity according to the configured rules. Define a separate set of rules for each resource.

In a relationship rule, you specify the **parent** and **child** resources. Entity lookup is performed using a single identifier: either by the parent resource or by the child resource.

Within a single rule, provide **only one** identifier. If both parent and child identifiers are specified, the lookup will be performed by the **parent** resource.

### Security

#### Approval before execution

You can require approvals for an action and set the number of required approvers. In this case, the action will not start until the specified number of approvals is received from the configured users.

The current number of approvals and the list of users whose approval is required are shown in the action runs table for the corresponding entity.

If approval is required from a specific user, they will receive a notification in the platform UI.

#### Account used to run the action

By default, access to external infrastructure services uses the credentials of the user who started the action. If needed, you can explicitly configure the action to run using the credentials of a specific account.

For actions that are triggered as automation events, specifying the account to run the action is required.

#### Credentials

For built-in actions, the platform predefines the required set of credentials. Their identifiers are loaded when you select the built-in backend. For each identifier, you must choose which credential type to use.

For webhook actions, you can reference credentials in the request body using `{{ .credentials.<credential_type_identifier> }}`.

## Action runs

### Action run records

Each time an action is started, a run record is created that includes the initiator, execution status, and a detailed log. Run records for each entity are available in the entity card on the **Action runs** tab. If approvals are required, they are collected on the same tab.

You can delete a run record or rerun the action. Rerunning creates a new run record.

### Run logging

For each action run, a database record is created containing the full execution log.

The `actions.logging.enabled` parameter in the DDP configuration file controls whether run logs are printed to stdout: when set to `true`, logs are printed; when set to `false`, they are not.

{{< alert level="info" >}}
Database records with the full action run log are created regardless of the `actions.logging.enabled` value.
{{< /alert >}}

### Action statuses

Each action run has a status record. Possible statuses are:

- **Created** — the run record is created, but the action has not started yet.
- **Unapproved** — the action is waiting for approvals.
- **Running** — the action is in progress.
- **Failed** — the action finished with an error.
- **Update failed** — the action completed, but updating the entity parameters failed.
- **Success** — the action completed successfully.
- **Retrying** — the action finished with an error and is being retried.
