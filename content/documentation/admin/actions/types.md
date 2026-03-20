---
title: Типы действий
---

## CreateDefectdojoEngagement

CreateDefectdojoEngagement — Creates a new engagement in the DefectDojo system. The action uses the DefectDojo API v2.

### Request Example

``yaml
name: example engagement
product: '1'
target_start: '2024-06-01'
target_end: '2024-06-30'
lead: '1'
``

### Request Specification

The list of fields corresponds to the official DefectDojo API, `/api/v2/engagements`. For more details, see the Defectdojo documentation [https://demo.defectdojo.org/api/v2/oa3/swagger-ui/].

### Credentials

* **token** — API v2 Key of the user on whose behalf the action will be executed.

## CreateDefectdojoProduct

CreateDefectdojoProduct — Creates a new product in the DefectDojo system. This action uses the DefectDojo API v2.

### Request Example

```yaml
name: example
description: example description
prod_type: 1
```

### Request Specification

The list of fields corresponds to the official DefectDojo API, `/api/v2/products`. For more details, see the Defectdojo documentation [https://demo.defectdojo.org/api/v2/oa3/swagger-ui/].

### Credentials

* **token** — API v2 Key of the user under whom the action will be executed.

## CreateGitlabBranches

CreateGitlabBranches — Creates new branches in the target repository.

### Request Example

```yaml
project_id: '0'
branches:
- branch: new-branch
ref: main
```

### Request Specification

| Name | Required | Description |
|-----------------|----------------|-----------------------------------------------------------------------------|
| project_id | Yes | Project ID in which to create branches |
| branches | Yes | List of branches to create |
| branches.branch | Yes | Name of the new branch |
| branches.ref | Yes | Name of an existing branch or SHA hash of a commit |

### Credentials

* **token** — the user token on whose behalf the action will be executed.

### Note

A valid GitLab token is required to execute the action. This token is passed through the credentials mechanism and used for authentication when calling the GitLab API (HTTP header 'Private-Token').

The action performs a POST request to the URL '/api/v4/projects/:id/repository/branches'. If the project is successfully created, GitLab returns information about the created branches.

## CreateGitlabGroupVariables

CreateGitlabGroupVariables — creates group-level variables in GitLab.

### Request Example

```yaml
group_id: '0'
variables:
- key: EXAMPLE_VARIABLE
value: value
```

### Request Specification

| Name | Required | Description |
|-----------------|----------------|-----------------------------------------------------------------------------|
| group_id | Yes | The group ID in which to create variables |
| variables | Yes | List of variables to create |

The list of variable fields corresponds to the official GitLab Group-level Variables API, `/groups/:id/variables`. For more details, see [the Gitlab documentation](https://docs.gitlab.com/api/group_level_variables/#create-variable).

### Credentials

* **token** — the token of the user under whom the action will be executed.

### Note

To execute the action, you must have valid credentials with a GitLab token. This token is passed through the credentials mechanism and is used for authentication when calling the GitLab API (HTTP header 'Private-Token').

The action makes a POST request to the URL: '/api/v4/groups/:id/variables'.

## CreateGitlabMergeRequest

CreateGitlabMergeRequest - creates a new Merge Request (MR) in the target repository. Files stored in the source repository are added to the Merge Request. The files can contain variables, the values ​​of which will be substituted when the MR is created.

### Request Example

```yaml
source_project_id: '0'
source_project_branch: example
source_project_tag: v1.0.0
target_project_id: '0'
merge_request_spec:
source_branch: example
target_branch: '1'
title: example
additionalIgnoreFiles:
- .ignore
- .example
values:
key1: value1
nested:
enabled: true
subkey: 123
```

### Request Specification

| Name | Required | Description | Default Value |
|-------------------------|----------------|--------------------------------------------------------------------------------------------------------------|-----------------------|
| source_project_id | Yes | ID of the project that serves as the source for the Merge Request | - |
| target_project_id | Yes | ID of the target project from which the Merge Request will be generated | - |
| merge_request_spec | Yes | Specification corresponding to the [GitLab Merge Requests API](https://docs.gitlab.com/ee/api/merge_requests.html#create-mr) | - |
| source_project_tag | No | Tag in the source project from which the Merge Request will be generated. If not specified, the branch in the source project is used. | - |
| source_project_branch | No | Branch in the source project from which the Merge Request will be generated. | main |
| additionalIgnoreFiles | No | List of files containing paths to exclude from the merge request. Populated similarly to [.templateignore](#templateignore) | - |
| values ​​| No | Variables used in templating, in the format `key: value` | - |

### Credentials

* **password** — the password (token) of the user under which the action will be executed.
* **username** — the username under which the action will be executed.

### Operation Algorithm

Platform:

1. Clones the template repository for generating the MR by its ID (**source_project_id**). More details about [templating](#operation-details).
1. Reads the `values.yaml` file stored in the repository root and defines default variables for templating.
1. Reads the variables passed when running the action and merges them with the variables from `values.yaml`. Priority is given to the variables passed when running the action.
1. Reads the `.templateignore` file and defines directories and files excluded from templating.
1. Renders files from templates, taking into account `values.yaml` and the variables passed to the action.
1. Changes the remote repository to the target one, according to its ID (**target_project_id**), and performs a git push to the target branch (**source_project_branch**) or to the main branch (**main**).
1. Creates a MR, according to the specified settings, by sending a POST request to the GitLab API.

### Note

This action requires a valid GitLab token. This token is passed through the credentials mechanism and is used for authentication when calling the GitLab API (HTTP header 'Private-Token').

This action makes a POST request to the URL: '/api/v4/projects/:id/merge_requests'.

## CreateGitlabProject

CreateGitlabProject — creates a new project in GitLab. This action calls the GitLab API to create a project with the specified parameters, such as the project name, path, description, and other settings. The GitLab token, which must be provided in the credentials, is used for authentication.

### Request Example

```yaml
name: example
path: example
description: example
default_branch: main
initialize_with_readme: false
namespace_id: '0'
``

### Request Specification

| Name | Required | Description | Default Value |
|-----------------------|----------------|--------------------------------------------------------------------------------------------|-----------------------|
| name | Yes | The name of the project to be created in GitLab | - |
| path | Yes | URL-compatible path of the project. Usually matches the name, but can differ | - |
| default_branch | Yes | The name of the branch to use by default, for example, "main" | - |
| namespace_id | Yes | The identifier of the namespace in GitLab in which the project will be created | - |
| initialize_with_readme | No | Flag that determines whether to initialize the project with a README file | false |
| description | No | Project description that will be visible to users | - |

### Credentials

* **token** — The token of the user under whom the action will be executed.

### Note

The action requires valid GitLab token credentials. The token containing the credentials is passed through the credentials mechanism and is used for authentication when calling the GitLab API (HTTP header 'Private-Token').

The action makes a POST request to the URL '/api/v4/projects', passing the request parameters in JSON format. If the project is successfully created, GitLab returns data about the newly created project.

## CreateGitlabProjectVariables

CreateGitlabProjectVariables — creates project-level variables in GitLab.

### Request Example

```yaml
project_id: '0'
variables:
- key: EXAMPLE_VARIABLE
value: value
```

### Request Specification

| Name | Required | Description |
|-----------------|----------------|-----------------------------------------------------------------------------|
| project_id | Yes | Project ID in which to create variables |
| variables | Yes | List of variables to create |

The list of variable fields corresponds to the official GitLab Project-level CI/CD variables API, `/projects/:id/variables`. For more details, see [the Gitlab documentation](https://docs.gitlab.com/api/project_level_variables/#create-a-variable).

### Credentials

* **token** — the user token under which the action will be executed.

### Note

This action requires valid GitLab token credentials. The token containing the credentials is passed through the credentials mechanism and used for authentication when calling the GitLab API (HTTP header 'Private-Token').

This action makes a POST request to the URL: '/api/v4/projects/:id/variables'.

## CreateGitlabProjectWebhook

CreateGitlabProjectWebhook — creates a webhook in a GitLab project.

### Request Example

```yaml
project_id: '0'
url: https://example.com
push_events: true
issues_events: true
merge_requests_events: true
pipeline_events: true
```

### Request Specification

| Name | Required | Description |
|----------------------|------------------|-----------------------------------------------------------|
| project_id | Yes | Project ID in which to create the webhook |
| url | Yes | Webhook URL |
| push_events | Yes | Trigger webhook when pushing to repository |
| issues_events | Yes | Trigger a webhook when an Issue is created |
| merge_request_events | Yes | Trigger a webhook when a Merge rRequest is created |
| pipeline_events | Yes | Trigger a webhook when a Pipeline starts |

### Credentials

* **token** — The token of the user under whom the action will be executed.

### Note

The action requires valid GitLab token credentials. The token containing the credentials is passed through the credentials mechanism and is used for authentication when calling the GitLab API (HTTP header 'Private-Token').

The action makes a POST request to the URL: '/api/v4/projects/:id/hooks'.

## CreateGitlabTag

CreateGitlabTag — creates a new tag in a GitLab project.

### Request Example

```yaml
project_id: '0'
tag_name: v1.0.0
ref: main
message: Tag description
```

### Request Specification

| Name | Required | Description |
|----------------------|----------------|------------------------------------------------------------------------------|
| project_id | Yes | Project ID in which to create the tag |
| tag_name | Yes | Tag name |
| ref | Yes | Branch name, tag, or SHA hash of the commit to which the new tag will refer |
| message | Yes | Tag description |

### Credentials

* **token** — the user token on whose behalf the action will be executed.

### Note

A valid GitLab token is required to execute the action. A token containing credentials is passed through the credentials mechanism and used for authentication when calling the GitLab API (HTTP header `Private-Token`).

The action is performed by a POST request to the URL: `/api/v4/projects/:id/repository/tags`.

## CreateGitlabRelease

CreateGitlabRelease — creates a GitLab release based on an existing tag.

### Request Example

```yaml
project_id: '0'
tag_name: v1.0.0
name: Release v1.0.0
description: |
## Changes:
- New feature.
- Bug fixes.
```

### Request Specification

| Name | Required | Description |
|--------------|----------------|-----------------------------------------------------------------------------|
| project_id | Yes | Project ID in which to create the release |
| tag_name | Yes | Name of an existing tag from which to base the release |
| name | Yes | Release name as displayed in GitLab |
| description | No | Release Description in Markdown Format |

### Credentials

* **token** — the token of the user under which the action will be executed.

### Note

The action requires valid GitLab token credentials. The token containing the credentials is passed through the credentials mechanism and used for authentication when calling the GitLab API (HTTP header 'Private-Token').

The action makes a POST request to the URL: '/api/v4/projects/:id/releases'.

## CreateKafkaACLs

CreateKafkaACLs — creates a new ACL set in Kafka.

### Example request

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
acls:
  - topics:
  - example_1
allow:
  - User:principal_2
  - Group:principal_3
deny:
  - User:principal_4
  - Group:principal_5
ops:
  - CREATE
  - READ
  - WRITE
  - DELETE
  - DESCRIBE
  - DESCRIBE_CONFIGS
  - ALTER
pattern: LITERAL
  - topics:
  - example_6
allow:
  - User:principal_7
allow_hosts:
  - 127.0.0.1
deny:
  - User:principal_8
deny_hosts:
  - 127.0.0.1
ops:
  - CREATE
pattern: LITERAL
```

### Request Specification

| Name | Required | Description | Possible Values ​​| Default |
|-------------------------|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|------------------------|
| securityProtocol | Yes | The protocol for connecting to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL | - |
| saslMechanism | No | The authentication mechanism that SASL will use. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. More details are available in the Kafka documentation [in the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 | - |
| acls | Yes | The set of ACLs to create | - | - |
| acls.ops | Yes | List of operations for which the rule will be created | [List of possible operations](#list-of-possible-operations) | - |
| acls.pattern | Yes | Pattern type | [List of possible patterns](#list-of-possible-patterns) | - |
| acls.topics | No | List of topic names for which the rule applies | - | - |
| acls.groups | No | List of group names for which the rule applies | - | - |
| acls.transactional_ids | No | List of transaction IDs for which the rule applies | - | - |
| acls.tokens | No | List of tokens for which the rule applies | - | - |
| acls.allow | No | List of hosts for which the operation is allowed | - | - |
| acls.deny | No | List of principals (user, group) for which the rule is denied | - | - |
| acls.deny_hosts | No | List of hosts for which the operation is denied                                                                                                                                                                      | -                                                       | -                      |

### Credentials

* **user** — the username under which the action will be executed.
* **password** — the password of the user under which the action will be executed.

### List of possible patterns

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

### List of possible operations

For more details, see the Kafka documentation [https://kafka.apache.org].

**Topic:**

* ALL.
* ALTER.
* ALTER_CONFIGS.
* CREATE.
* DELETE.
* DESCRIBE.
* DESCRIBE_CONFIGS.
* READ.
* WRITE.

**Group:**

* ALL.
* DELETE.
* DESCRIBE.
* READ.

**TransactionalID:**

* ALL.
* DESCRIBE.
* WRITE.

**Tokens:**

* DESCRIBE.

## CreateKafkaTopics

CreateKafkaTopics — creates new topics in Kafka.

### Request Example

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
partitions: 1
replication_factor: 1
configs: {}
topics:
  - example_1
  - example_2
```

### Request Specification

| Name | Required | Description | Possible Values ​​|
|-------------------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| securityProtocol | Yes | The protocol for connecting to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
| saslMechanism | No | The authentication mechanism that will use SASL. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
| partitions | Yes | The number of partitions into which the topic will be divided | - |
| replication_factor | Yes | The number of copies (replicas) of each partition of the topic to be placed on different brokers | - |
| configs | Yes | Key-value configuration for created topics | Values ​​are provided [in the Kafka documentation](https://kafka.apache.org/documentation/#topicconfigs) |
| topics | Yes | List of topic names to create | - |

### Credentials

* **user** — The username under which the action will be executed.
* **password** — The password of the user under which the action will be executed.

## CreateKafkaUsers

CreateKafkaUsers — creates new SASL/SCRAM users in Kafka.

### Request Example

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
users:
- user: example_user_1
password: example_password_user_1
mechanism: SCRAM-SHA-256
iterations: 4096
- user: example_user_2
password: example_password_user_2
mechanism: SCRAM-SHA-256
iterations: 4096
```

### Request Specification

| Name | Required | Description | Possible Values ​​|
|-------------------|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| securityProtocol | Yes | Protocol for connecting to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
| saslMechanism | No | Authentication mechanism that will use SASL. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
| users | Yes | Set of users to create | - |
| users.user | Yes | Name of the user to create | - |
| users.password | Yes | Password of the user to create | - |
| users.mechanism | Yes | Authentication mechanism for the created user | SCRAM-SHA-256, SCRAM-SHA-512 |
| users.iterations | Yes | Number of iterations to use for password hashing                                                                                                                                     | От 4096 до 16384                        |

### Credentials

* **user** — the username that will run the action.
* **password** — the password of the user that will run the action.

## CreateCodeScoringProject

CreateCodeScoringProject — creates a new project in the CodeScoring system.
The action uses the CodeScoring API to register a project with the specified parameters: project name, repository URL, VCS system ID, and the option to automatically run SCA analysis after cloning the repository.

### Example Request

```yaml
name: example-project
repository: https://gitlab.example.com/group/project.git
vcs_id: 2
run_sca_after_clone: ​​true
```

### Request Specification

| Name | Required | Description |
|--------------------|----------------|------------------------------------------------------------------------------|
| name | Yes | Project name in CodeScoring |
| repository | Yes | Repository URL (e.g., <https://gitlab.example.com/group/project.git>) |
| vcs_id | Yes | VCS system ID in CodeScoring (must be greater than 0) |
| run_sca_after_clone| No | Automatically run SCA analysis after cloning the repository |

### Response

The response returns the created project object with the following information: project ID (pk), name, project type, description, repository information, project status, access rights, license, number of dependencies and vulnerabilities, project languages, scan schedule status, and the dates of the first and last SCA scan.

## DeleteCodeScoringProject

DeleteCodeScoringProject — deletes a project in the CodeScoring system by its ID.

### Request example

```yaml
id: 1
```

### Request specification

| Name | Required | Description |
|----------|----------------|-----------------------------|
| id | Yes | Project ID in CodeScoring |

## CreateKeycloakClient

CreateKeycloakClient — Creates a new client in Keycloak.

### Request Example

```yaml
realm: master
config:
clientId: example
name: example
enabled: true
clientAuthenticatorType: client-secret
secret: secret
defaultClientScopes:
- roles
- profile
- email
optionalClientScopes:
- address
- phone
- offline_access
```

### Request Specification

| Name | Required | Description | Possible Values ​​|
|---------|----------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| realm | Yes | Realm in Keycloak where the client should be created | - |
| config | Yes | Parameters of the created client according to Keycloak's ClientRepresentation specification | [Documentation](https://www.keycloak.org/docs-api/latest/rest-api/index.html#ClientRepresentation) |

### Credentials

* **username** — The username of the user who will run the action.
* **password** — The password of the user who will run the action.

## CreateKubernetesResource

CreateKubernetesResource — Creates a new resource or resources in a Kubernetes cluster or updates existing ones.

### Request Example

```yaml
manifests:
- apiVersion: v1
kind: Namespace
metadata:
name: example1
- apiVersion: v1
kind: Namespace
metadata:
name: example2
```

### Request Specification

| Name | Required | Description |
|----------------------------|----------------|------------------------------------------------------------------------------------|
| manifests | Yes | Kubernetes manifests to be applied |

### Credentials

* **token** — Kubernetes service account token.

## CreateRepositoryFromTemplate

CreateRepositoryFromTemplate — creates a new repository from a template in Gitlab. The rendering engine is based on the [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) and supports all built-in methods, as well as extensions added to the platform.

### Request Example

```yaml
sourceBranch: main
sourceTag: v1.0.0
templateRepositoryUrl: https://gitlab.example.com/example-1.git
targetRepositoryUrl: https://gitlab.example.com/example-2.git
targetBranch: master
additionalIgnoreFiles:
- .ignore
- .example
values:
key1: value1
nested:
enabled: true
subkey: 123
```

### Request Specification

| Name | Required | Description | Default Value |
|--------------------------|----------------|-------------------------------------------------------------------------------------------------------------------------|-----------------------|
| templateRepositoryUrl | Yes | URL of the template repository | - |
| targetRepositoryUrl | Yes | URL of the repository that will be created as a result of the action | - |
| values ​​| Yes | Variables used in templating, in the format `key: value` | - |
| additionalIgnoreFiles | No | List of files containing paths to exclude from the target repository. Populated similarly to [.templateignore](#templateignore) | - |
| sourceTag | No | Tag of the template repository that will be used for templating. If not specified, the template repository branch is used | - |
| sourceBranch | No | Branch of the template repository that will be used for templating | main |
| targetBranch | No | Branch of the target repository that will be created as a result of the action | main |

### Credentials

* **password** — password (token) of the user under whom the action will be executed.
* **username** — the username that will run the action.

### Operation Algorithm

Platform:

1. Clone the template repository at the specified URL (**templateRepositoryUrl**), using either **sourceTag**, **sourceBranch**, or the **main** branch as the ref.
1. Reads the `values.yaml` file stored in the repository root and defines default variables for templating.
1. Reads the variables passed when running the action and merges them with the variables from `values.yaml`. Priority during merging is given to the variables passed when running the action.
1. Reads the `.templateignore` file and defines directories and files to exclude from templating.
1. Renders files from templates, taking into account `values.yaml` and the variables passed to the action.
1. Changes the remote repository to the target one (**targetRepositoryUrl**) and does a git push to the target branch (**targetBranch**), or to the main branch **main**.

### Operation Details

This action supports templating of directory and file names. To do this, add an expression in the [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) format to their names.
For example, the directory `src/{{ .module }}/utils` with a value of **module** and a value of **example** will be rendered to the directory `src/example/utils` in the target repository.

If the file content is missing after rendering from the template, the file is not created. For example, a file with the following contents:

```go
{{- if .createContent }}
- This is the content that will be displayed if the variable createContent == true
{{- end }}
```

will not be created if the variable **createContent** is **false**. Similarly, files that are initially empty will not be created.

If the templating variables are missing both in the `values.yaml` file and in the variables passed when running the action, rendering will fail and the target repository will not be created.

### Template Repository Variables

To add default variables used for templating, you must create a values.yaml file with the appropriate contents in the root of the repository.

Example of a `values.yaml` file:

```yaml
module: example
createContent: false
```

The `values.yaml` file is optional.

<a id="templateignore"></a>

### Excluding Files

Some files may contain variables in the [Go template](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax) format that must be preserved when rendering a repository from a template. For example, Helm charts in the `helm` directory. The `.git` directory is always ignored.

To exclude such files from the rendering engine, add a `.templateignore` file with the appropriate contents to the root of the repository.

Example of a `.templateignore` file to ignore the contents of the `helm` and `docs` directories:

```sh
helm/**
docs/**
```

The `.templateignore` file is optional.

### Example directory structure of a template repository

```sh
├── example-folder-01
│ ├── example-file-01
│ └── {{ .example }}-file-02
├── {{ .example }}-folder-02
│ └── ...
├── values.yaml
└── .templateignore
```

If the **example** variable is set to **new** when rendering the repository, the final structure after rendering will look like this:

```sh
├── example-folder-01
│ ├── example-file-01
│ └── new-file-02
├── new-folder-02
│ └── ...
├── values.yaml
└── .templateignore
```

### Local Debugging

The **ddp-render-dir** utility is available for local debugging of templates.

Utility:

1. Creates a copy of the source directory.
1. Renders files in this directory using the same rules as for creating repositories from templates.

Command line switches:

* **--source-dir** — the source directory to render.
* **--target-dir** — the directory where the rendered result will be placed.
* **--values** (optional) — path to the `values.yaml` file containing variables to be used during rendering.
* **--ignore-files** (optional) — list of files containing paths to exclude from the target repository.

## CreateSonarqubeProject

CreateSonarqubeProject — Creates a new project in SonarQube.
This action uses the SonarQube Web API to create a project with the specified parameters, such as the project key, project name, master branch, new code definition parameters, and project visibility. Authentication is performed using the SonarQube token, which must be passed in the credentials.

### Request Example

```yaml
project: example-project
name: example-project
mainBranch: develop
newCodeDefinitionType: NUMBER_OF_DAYS
newCodeDefinitionValue: '30'
visibility: public
```

### Request Specification

| Name | Required | Description | Possible Values ​​| Default Value |
|-------------------------|----------------|-------------------------------------------------------------------------------------------------|----------------------------------------------------|------------------------|
| project | Yes | Unique project identifier (Project Key) in SonarQube | - | - |
| name | Yes | Project name displayed in the SonarQube interface | - | - |
| mainBranch | No | Name of the main project branch | - | master |
| newCodeDefinitionType | No | Method for defining "new code" | PREVIOUS_VERSION, NUMBER_OF_DAYS, REFERENCE_BRANCH | - |
| newCodeDefinitionValue | No | Value for defining "new code" (for example, the number of days if the type is NUMBER_OF_DAYS) | - | - |
| visibility | No | Project visibility | private, public | private |

### Credentials

* **token** — a Sonarqube token of the User Token type, generated by the user on whose behalf the action will be executed.

## CreateVaultSecret

CreateVaultSecret — Creates a secret with one or more values ​​in HashiCorp Vault.

### Request Example

```yaml
path: example/data/path
secrets:
- key: key1
value: value1
- key: key2
value: value2
```

### Request Specification

| Name | Required | Description | Possible Values ​​| Default |
|---------------------------|----------------|-----------------------------------------------------------------|-------------------|--------------|
| path | Yes | Path where the secret will be stored in Vault | | - |
| allow_update | No | Determines the action's behavior if an existing secret exists | true, false, merge | false |
| secrets | Yes | Set of secrets to create in Vault | | - |
| secrets.key | Yes | Secret name (ID) | | - |
| secrets.value | Yes | Secret value | | - |

### Credentials

* **token** — A Vault token that has the necessary permissions to create secrets.

### Note

**allow_update** – Determines the action's behavior when an existing secret exists:

- `false` (default) – the action fails if the secret already exists;
- `true` – a new version of the secret is created with the values ​​passed in the action;
- `merge` – only the secret keys specified in the action are updated or created. Existing keys not mentioned in the action are preserved unchanged.

## DeleteVaultSecret

DeleteVaultSecret — Deletes a secret from HashiCorp Vault.

### Request Example

```yaml
path: example/data/path
```

### Request Specification

| Name | Required | Description |
|---------------------------|----------------|------------------------------------------------------------------------------------|
| path | Yes | Path where the secret in Vault to be deleted is located |

### Credentials

* **token** — A Vault token that has the necessary permissions to delete secrets.

## DeleteDefectdojoProduct

DeleteDefectdojoProduct — Deletes a product from DefectDojo. This action uses DefectDojo API v2.

### Request Example

```yaml
id: 1
```

### Request Specification

| Name | Required | Description | Default value |
|-------------|----------------|----------------------------------------------------|------------------------|
| id | Yes | ID of the product to delete | - |

### Credentials

* **token** — API v2 Key of the user under which the action will be executed.

## DeleteGitlabProject

DeleteGitlabProject - deletes an existing project in GitLab. This action calls the GitLab API to delete the project. Authentication uses the GitLab token, which must be provided in the credentials.

### Request example

```yaml
project_id: 0
```

### Request specification

| Name | Required | Description |
|---------------------------|----------------|------------------------------------------------------------------------------------|
| project_id | Yes | ID of the project to be deleted |

### Credentials

* **token** — the token of the user under whom the action will be executed.

### Note

To execute the action, you must have valid credentials with a GitLab token.
This token is passed through the credentials mechanism and is used for authentication when calling the GitLab API (HTTP header `Private-Token`).

The action makes a DELETE request to the URL: `/api/v4/projects/:id`.

## DeleteKafkaACLs

DeleteKafkaACLs — deletes a set of ACLs in Kafka.

### Example request

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
acls: 
- topics: 
- example_1 
allow: 
- User:principal_2 
- Group:principal_3 
allow_hosts: 
- 127.0.0.1 
deny: 
- User:principal_4 
- Group:principal_5 
deny_host: 
- 127.0.0.1 
ops: 
- CREATE 
- READ 
- WRITE 
- DELETE 
- DESCRIBE 
- DESCRIBE_CONFIGS 
- ALTER 
pattern: LITERAL 
- any_topic: true 
any_group: true 
any_transactional_id: true 
any_allow: true 
any_allow_hosts: true 
any_deny: true 
any_deny_hosts: true 
ops: 
- ANY 
pattern: ANY 
- any_resource: true 
any_allow: true 
any_allow_hosts: true 
any_deny: true 
any_deny_hosts: true 
ops: 
- ANY 
pattern: ANY
```

### Request specification

| Name | Required | Description | Possible values ​​| Default value |
|---------------------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|-------------------------|
| securityProtocol | Yes | The protocol for connecting to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL | - |
| saslMechanism | No | The authentication mechanism that SASL will use. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. More details are available in the Kafka documentation [in the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 | - |
| acls | Yes | The set of ACLs to create | - | - |
| acls.ops | Yes | The list of operations for which the rule will be created. | [List of possible operations](#list-of-possible-operations) | - |
| acls.pattern | Yes | Pattern type | [List of possible patterns](#list-of-possible-patterns) | - |
| acls.any_resource | No | All resources | - | false |
| acls.topics | No | A list of topic names to apply the rule to | - | - |
| acls.any_topic | No | All topics | - | false |
| acls.groups | No | List of group names to apply the rule to | - | - |
| acls.any_group | No | All topics | - | false |
| acls.transactional_ids | No | List of transaction IDs to apply the rule to | - | - |
| acls.any_transactional_id | No | Any transaction | - | false |
| acls.tokens | No | List of tokens to apply the rule to | - | - |
| acls.any_token | No | All tokens | - | false |
| acls.allow | No | List of principals (user, group) to allow the rule for | - | - |
| acls.any_allow | No | Any principal (user, group) | - | false |
| acls.allow_hosts | No | List of hosts to allow the operation for | - | - |
| acls.any_allow_hosts | No | Any host from which the operation is allowed | - | false |
| acls.deny | No | List of principals (user, group) to deny the rule for | - | - |
| acls.any_deny | No | Any principal (user, group) | - | false |
| acls.deny_hosts | No | List of hosts for which to deny the operation | - | - |
| acls.any_deny_hosts | No | Any host from which the operation is prohibited | - | false |

### Credentials

* **user** — the username under which the action will be executed.
* **password** — the password of the user under which the action will be executed.

### List of possible patterns

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

### List of possible operations

Detailed descriptions are available in the Kafka documentation [in the Kafka documentation](https://kafka.apache.org/39/documentation/#operations_resources_and_protocols)

**Topic:**

* ALL.
* ALTER.
* ALTER_CONFIGS.
* CREATE.
* DELETE.
* DESCRIBE.
* DESCRIBE_CONFIGS.
* READ.
* WRITE.

**Group:**

* ALL.
* DELETE. * DESCRIBE.
* READ.

**TransactionalID:**

* ALL.
* DESCRIBE.
* WRITE.

**Token:**

* DESCRIBE.

## DeleteKafkaTopics

DeleteKafkaTopics — deletes existing topics in Kafka.

### Request example

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
topics:
  - example_1
  - example_2
```

### Request Specification

| Name | Required | Description | Possible Values ​​|
|---------------------------|----------------|-----------------------------------------------------|-----------------------------------------|
| auth_type | Yes | Kafka Authorization Type | PLAINTEXT, SCRAM-SHA-256, SCRAM-SHA-512 |
| topics | Yes | List of topic names to delete | - |

### Credentials

* **user** — The username under which the action will be executed.
* **password** — The password of the user under which the action will be executed.

## DeleteKafkaUsers

DeleteKafkaUsers — Deletes existing SASL/SCRAM users in Kafka.

### Request Example

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
users:
- user: example_user_1
mechanism: SCRAM-SHA-256
- user: example_user_2
mechanism: SCRAM-SHA-256
```

### Request Specification

| Name | Required | Description | Possible Values ​​|
|-------------------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| securityProtocol | Yes | Protocol for connecting to Kafka. More details can be found [in the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
| saslMechanism | No | The authentication mechanism that will use SASL. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see the Kafka documentation [https://kafka.apache.org/documentation/#security_sasl_mechanism] | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
| users | Yes | The set of users to delete | - |
| users.user | Yes | The name of the user to delete | - |
| users.mechanism | Yes | The authentication mechanism of the user to delete | SCRAM-SHA-256, SCRAM-SHA-512 |

### Credentials

* **user** — The username that will run the action.
* **password** — The password of the user that will run the action.

## DeleteKubernetesResource

DeleteKubernetesResource — Deletes an existing resource in the Kubernetes cluster.

### Request Example

```yaml
group: apps
version: v1
resource_type: deployments
resource_name: nginx-deployment
namespace: example
```

### Request Specification

| Name | Required | Description | Possible Values                                                                                                                                                                                                                                                                                                                                                                                                                         |
|---------------------------|-----------------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| group | Yes | Resource API Group. Specifies the API group to which the object being deleted belongs. | [Defining required Group and Version](#determining-the-required-group-and-version)                                                                                                                                                                                                                                                                                                                                                      |
| version | Yes | Resource API Version | [Defining required Group and Version](#determining-the-required-group-and-version)                                                                                                                                                                                                                                                                                                                                                             |
| resource_type | Yes | The type of resource being deleted | pods, services, deployments, statefulsets, daemonsets, replicasets, jobs, cronjobs, nodes, namespaces, configmaps, secrets, persistentvolumes, persistentvolumeclaims, limitranges, resourcequotas, horizontalpodautoscalers, ingresses, networkpolicies, serviceaccounts, roles, clusterroles, rolebindings, clusterrolebindings, podsecuritypolicies, storageclasses, volumeattachments, events, endpoints, customresourcedefinitions |
| resource_name | Yes | Name of the specific resource to be deleted | -                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Namespace | Yes | Namespace in which the resource is located | -                                                                                                                                                                                                                                                                                                                                                                                                                                       |

### Credentials

* **token** — the Kubernetes service account token.

### Determining the required Group and Version

Each resource type has its own API Group and Version.
A full list of API resources with their Groups and Versions is provided [in the Kubernetes documentation](https://kubernetes.io/docs/reference/kubernetes-api/).

If you don't know the required Group and Version, you can use the current values.
There are several ways to determine them:

#### Using the kubectl utility

The `kubectl explain` command displays the `apiVersion` for the resource.

Example:

```bash
kubectl explain deployment
```

Output:

```yaml
GROUP: apps
KIND: Deployment
VERSION: v1

DESCRIPTION:
Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
...
```

#### Using the documentation

How to search the documentation:

1. Find the desired resource (e.g., Deployment).
1. The header will indicate the API Group and version. Example for Deployment:

```yaml
apiVersion: apps/v1
```

Here:
* **API Group** - apps
* **Version** - v1

## DeleteSonarqubeProject

DeleteSonarqubeProject — deletes a project from SonarQube.

### Request Example

```yaml
project: example-project
```

### Request Specification

| Name | Required | Description | Default Value |
|------------------|------------------|----------------------------------------------------------------|------------------------|
| project | Yes | Project Key of the project to be deleted | - |

### Credentials

* **token** — Sonarqube User Token generated by the user on whose behalf the action will be executed.

## DeleteSonarqubeProjects

DeleteSonarqubeProjects — deletes one or more projects from SonarQube.

### Request Example

```yaml
analyzedBefore: 2017-10-19T13:00:00+0200
onProvisionedOnly: 'false'
projects: example_project,another_project
q: example
qualifiers: TRK
```

### Request Specification

| Name | Required | Description | Possible Values ​​| Example Values ​​|
|--------------------|------------------|---------------------------------------------------------------------------------|------------------------------|---------------------------------------|
| analyzedBefore | No | Delete all projects for which the last analysis is older than a certain date | - | 2017-10-19, 2017-10-19T13:00:00+0200 |
| onProvisionedOnly | No | Filter projects by the `onProvisionedOnly` value | true, false, yes, no | - |
| projects | No | List of project keys to delete | - | my_project, another_project |
| q | No | Delete all projects whose name or key contains the specified substring | - | example |
| qualifiers | No | Filter projects by the specified qualifiers | TRK, VW, APP | |

### Credentials

* **token** — a Sonarqube token of the User Token type, generated by the user on whose behalf the action will be executed.

## StartGitlabPipeline

StartGitlabPipeline — Starts pipeline execution in GitLab.

### Request Example

```yaml
project_id: 0
ref: main
variables:
- key: example-key
value: example-value
```

### Request Specification

| Name | Required | Description |
|---------------------------|------------------|-----------------------------------------------------------------------------|
| project_id | Yes | ID of the project in which to run the pipeline |
| ref | Yes | Branch name, tag, or SHA hash of the commit on which to run the pipeline |
| variables | No | List of variables to pass to the pipeline being launched |
| variables.key | Yes | Variable name |
| variables.value | Yes | Variable value |

### Credentials

* **token** — the token of the user under whose name the action will be executed.

### Note

To execute the action, you must have valid credentials with the GitLab token.
This token is passed through the credentials mechanism and is used for authentication when calling the GitLab API (HTTP header `Private-Token`).

The action makes a POST request to the URL: `/api/v4/projects/:id/pipeline`.

## CreateVaultKubernetesAuthRole

CreateVaultKubernetesAuthRole — Creates or updates a Kubernetes authentication role in HashiCorp Vault.

### Request Example

```yaml
mountPath: kubernetes
role: example
bound_service_account_names:
- default
bound_service_account_namespaces:
- default
optional:
token_ttl: 1h
token_max_ttl: 12h
audience: vault
token_policies:
- default
```

### Request Specification

| Name | Required | Description |
|-------------------------------------|----------------|----------------------------------------------------------------------------------|
| mountPath | required | Mount path of the Kubernetes auth backend in Vault (e.g., kubernetes) |
| role | required | The name of the role being created in Vault |
| bound_service_account_names | required | List of service account names allowed through this role |
| bound_service_account_namespaces | required | List of namespaces allowed through this role |
| optional | optional | Additional role parameters (listed in the following table) |

Supported values ​​in optional:

| Field | Type | Description |
|-------------------------|--------------|-----------------------------------------------------------------------|
| token_ttl | string | Time to live (TTL) of the token issued upon login |
| token_max_ttl | string | Maximum token TTL |
| token_policies | []string | Additional policies assigned upon login |
| audience | string | The JWT audience (aud) value that Vault expects for the token |
| token_period | string | Token issuance frequency |
| token_explicit_max_ttl | string | Explicit upper limit on the token TTL |
| token_num_uses | int | Limit on the number of token uses |
| token_type | string | The type of token being issued (e.g., service, batch) |
| alias_name_source | string | Source alias name for identity |
| token_no_default_policy | bool | Exclude default policy from the token |
| token_bound_cidrs | []string | Limit CIDR ranges within which the issued token can be used |

A full list of supported parameters is provided in the [official documentation](https://developer.hashicorp.com/vault/docs/auth/kubernetes#parameters) of HashiCorp Vault.

### Credentials

* **token** — A Vault token with permissions to create/update Kubernetes auth backend roles.

## CreateNexusRepository

**CreateNexusRepository** — Creates a new repository of any supported type (Maven, Docker, NPM, etc.) in Nexus Repository Manager 3 using the REST API.

The format, type, and other key settings are fully configurable and comply with the Nexus API.

### Request example (Maven hosted)

```yaml
description: |
Maven hosted repo for internal Java build artifacts.
name: my-maven-repo
format: maven
type: hosted
online: true
storage: 
blobStoreName: default 
strictContentTypeValidation: true 
writePolicy: ALLOW
cleanup: 
policyNames: 
-maven-cleanup
maven: 
versionPolicy: RELEASE 
layoutPolicy: PERMISSIVE
```

### Example request (Docker group)

```yaml
description: |
  Docker group repo aggregating hosted+proxy.
name: my-docker-group
format: docker
type: group
online: true
storage:
blobStoreName: default
strictContentTypeValidation: true
group:
memberNames:
  - docker-hosted
  - docker-proxy
docker:
v1Enabled: false
forceBasicAuth: true
httpPort: 5001
```

### Request Specification

| Field | Required | Description | Example                                  |
|------------|-----------------|---------------------------------------------------------------------------------------------------|------------------------------------------|
| `description` | No | Documentation on the purpose of this action/repository. Not used by Nexus itself, for the UI only | -                                        |
| `name` | Yes | The name of the repository being created. Must be unique within Nexus | my-maven-repo                            |
| `format` | Yes | Format (`maven`, `docker`, `npm`, `raw`, etc.) | maven                                    |
| `type` | Yes | Type: `hosted`, `proxy`, or `group` | hosted                                   |
| `online` | Yes | Whether the repository is accessible (`true`/`false`) | true                                     |
| `storage` | Yes | Storage object: `blobStoreName`, `strictContentTypeValidation`, `writePolicy` | [Example](#request-example-maven-hosted) |
| `cleanup` | None | Associated cleanup policies (`policyNames`) | policyNames: [maven-cleanup]             |
| `maven` | for maven | Maven-only: `versionPolicy`, `layoutPolicy` | [Example](#request-example-maven-hosted) |
| `proxy` | for proxy | Proxy repository: `remoteUrl`, `contentMaxAge`, `metadataMaxAge` | -                                        |
| `group` | for group | List of included memberNames | [Example](#request-example-docker-group)              |
| `docker` | for docker | Docker-specific: `httpPort`, `v1Enabled`, `forceBasicAuth` | [Example](#request-example-docker-group) |
| `component` | very rare | Only for some non-standard scenarios | -                                        |
| `attributes`| None | Any custom fields | -                                        |

### Requirements

- Use only those blocks (`maven`, `group`, `proxy`, `docker`, etc.) that are supported for your type/format.
- For Maven hosted, `maven: {versionPolicy, layoutPolicy}` is required.
- For group, `group.memberNames` is required.
- For proxy, `proxy.remoteUrl` is required.
- For Docker, use the specific fields in `docker`.

### Credentials

* **token** — a base64 string (`admin:password`), used as Basic Auth for requests to Nexus.

## DeleteNexusRepository

**DeleteNexusRepository** — deletes an existing repository from Nexus Repository Manager 3.

### Request Example

```yaml
name: my-repo-to-delete
```

### Request Specification

| Field | Required | Description |
|------|-----------------|------------------------------------------|
| `name` | Yes | Name of the repository to delete |

### Algorithm

- A `DELETE` operation is performed on `/service/rest/v1/repositories/{name}`, where `{name}` is the value of the `name` field. - If the repository is found and deleted, a 204 response is returned.
- If not found, a 404 error is returned.

### Credentials

* **token** — a base64 string (`admin:password`), used as Basic Auth for requests to Nexus.

## CreateNexusPrivilege

CreateNexusPrivilege — creates a new privilege in Nexus Repository Manager 3. Privileges define access rights to repositories and other Nexus resources.

### Example request (repository-view)

```yaml
name: example-privilege
description: Example privilege description
type: repository-view
actions: 
- READ 
- BROWSE
format:maven2
repository: maven-releases
```

### Example request (repository-content-selector)

```yaml
name: content-selector-privilege
description: Privilege with content selector
type: repository-content-selector
actions: 
- READ
format:maven2
repository: maven-releases
contentSelector: my-content-selector
```

### Request Example (wildcard)

```yaml
name: wildcard-privilege
description: Wildcard privilege
type: wildcard
pattern: nx-*
actions:
- READ
```

### Request Specification

| Field | Required | Description |
|------------------|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name | Yes | Name of the privilege being created. Must be unique within Nexus |
| description | No | Privilege description |
| type | Yes | Privilege type: `repository-view`, `repository-content-selector`, `repository-admin`, `application`, `wildcard` |
| actions | No | List of actions allowed by the privilege (e.g., `READ`, `BROWSE`, `CREATE`, `UPDATE`, `DELETE`) |
| format | None | Repository format (e.g., `maven2`, `docker`, `npm`). Used for `repository-view`, `repository-content-selector`, and `repository-admin` types |
| repository | None | Repository name. Used for `repository-view`, `repository-content-selector`, and `repository-admin` types |
| contentSelector | None | Content selector name. Required for `repository-content-selector`. If omitted or invalid, the type is automatically converted to `repository-view` |
| pattern | None | Pattern for `wildcard` type |
| domain | None | Domain for `application` type |
| attributes | None | Additional attributes in key-value format |

### Note

For the `repository-content-selector` type, the content selector must exist in Nexus before creating the privilege. If the content selector is not specified or is invalid, the action automatically converts the privilege type to `repository-view`.

### Credentials

* **token** — base64 string (`admin:password`), used as Basic Auth for requests to Nexus.

## AssignNexusPrivilege

AssignNexusPrivilege — assigns privileges to an existing role in Nexus Repository Manager 3. The action gets the current role configuration and merges the existing privileges with the new ones.

### Request Example

```yaml
roleId: example-role
privileges:
- example-privilege
- another-privilege
```

### Request Specification

| Field | Required | Description |
|------------|----------------|------------------------------------------------------------------------------|
| roleId | Yes | ID of the role to which privileges are being assigned |
| privileges | Yes | List of privilege names to assign to the role |

### Operation Algorithm

1. Gets the current role configuration from Nexus.
2. Merges the role's existing privileges with the new privileges from the request.
3. Updates the role with the merged list of privileges.

### Credentials

* **token** — base64 string(`admin:password`), used as Basic Auth for requests to Nexus.

### Note

The role must exist in Nexus before privileges can be assigned. If the role is not found, the action will fail. All specified privileges must also exist in Nexus.

## DeleteNexusPrivilege

DeleteNexusPrivilege — Removes a privilege from Nexus Repository Manager 3.

### Request Example

```yaml
name: example-privilege
```

### Request Specification

| Field | Required | Description |
|------|-----------------|--------------------------------------------|
| name | Yes | Name of the privilege to remove |

### Credentials

* **token** — base64 string(`admin:password`), used as Basic Auth for requests to Nexus.

## CreateNexusRole

CreateNexusRole — Creates a new role in Nexus Repository Manager 3. Roles group privileges and can include other roles.

### Request Example

```yaml
id: example-role
name: Example Role
description: Example role description
privileges:
- nx-repository-view-*-*-read
- nx-repository-view-maven2-*-browse
roles: []
```

### Request Specification

| Field | Required | Description |
|-------------|-----------------|------------------------------------------------------------------------------|
| id | Yes | Unique role identifier |
| name | Yes | Role name |
| description | No | Role description |
| privileges | No | List of privilege names assigned to the role |
| roles | No | List of IDs of other roles that are included in this role |

### Credentials

* **token** — a base64 string (`admin:password`), used as Basic Auth for requests to Nexus.

## AssignNexusRole

AssignNexusRole — assigns roles to an existing user in Nexus Repository Manager 3. This action retrieves the user's current configuration and merges existing roles with new ones.

### Request Example

```yaml
userId: example-user
roles:
- example-role
- another-role
```

### Request Specification

| Field | Required | Description |
|--------|-----------------|--------------------------------------------------------------|
| userId | Yes | ID of the user to assign roles to |
| roles | Yes | List of role IDs to assign to the user |

### Operation Algorithm

1. Retrieves the current user configuration from Nexus.
2. Merge the user's existing roles with the new roles from the request.
3. Updates the user with the combined list of roles.

### Credentials

* **token** — base64 string (`admin:password`), used as Basic Auth for requests to Nexus.

### Note

The user must exist in Nexus before assigning roles. If the user is not found, the action will fail. All specified roles must also exist in Nexus.

## DeleteNexusRole

DeleteNexusRole — Deletes a role from Nexus Repository Manager 3.

### Request Example

```yaml
id: example-role
```

### Request Specification

| Field | Required | Description |
|------|-----------------|-----------------------------------------|
| id | Yes | ID of the role to delete |

### Credentials

* **token** — base64 string (`admin:password`), used as Basic Auth for requests to Nexus.

## CreateNexusUser

CreateNexusUser — Creates a new user in Nexus Repository Manager 3.

### Request Example

```yaml
userId: example-user
firstName: First
lastName: Last
emailAddress: user@example.com
password: password
status: active
roles:
- nx-admin
```

### Request Specification

| Field | Required | Description |
|-------------|-----------------|------------------------------------------------------------------------------|
| userId | Yes | Unique user identifier |
| firstName | Yes | Username |
| lastName | Yes | User's last name |
| emailAddress| Yes | User's email address |
| password | Yes | User's password |
| status | Yes | User status: `active` or `disabled` |
| roles | None | List of role IDs assigned to the user upon creation |

### Credentials

* **token** — base64 string (`admin:password`), used as Basic Auth for requests to Nexus.

## DeleteNexusUser

DeleteNexusUser — Deletes a user from Nexus Repository Manager 3.

### Request Example

```yaml
userId: example-user
```

### Request Specification

| Field | Required | Description |
|--------|-----------------|--------------------------------------------|
| userId | Yes | ID of the user to delete |

### Credentials

* **token** — base64 string(`admin:password`), used as Basic Auth for requests to Nexus.

## Wait

Wait — waits for the specified number of seconds (with an optional random addition called jitter). Intended for use in processes as a delay element, for example, to wait for the results of a previous action to be applied.

### Request Example

```yaml
duration_seconds: 10
max_jitter_seconds: 0
description: "Waiting for release"
```

### Request Specification

| Name | Required | Description | Default Value |
|---------------------|----------------|----------------------------------------------------------------------------------|-----------------------|
| duration_seconds | Yes | Base wait duration in seconds (0–86400, i.e., up to 24 hours) | - |
| max_jitter_seconds | No | Maximum random wait time in seconds (0–N). 0 — jitter disabled | 0 |
| description | No | Description to display in logs and action response | - |
