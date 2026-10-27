---
title: Datasource types
description: Built-in DDP datasource types — authorization, response specification, configuration, and parameters.
weight: 20
---

## DefectdojoProducts

A datasource of the `DefectdojoProducts` type returns a list of products in Defectdojo.

### Authorization

Authorization configuration is described in the ["DefectDojo external service"](../external-services/#defectdojo) section.

### Response specification

Deckhouse Development Portal (DDP) sends a GET request to the following URL: `/api/v2/products`.
All available values are returned.
[Response specification](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).

### Configuration

In the `URL` parameter, specify the DefectDojo URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## GitlabGroups

A datasource of the `GitlabGroups` type returns a list of groups in GitLab.

### Authorization

Authorization configuration is described in the ["GitLab external service"](../external-services/#gitlab) section.

### Response specification

The portal sends a GET request to the GitLab API at the following URL: `/api/v4/groups`.
The portal returns all available values.
[Response specification](https://docs.gitlab.com/api/groups/#list-groups).

### Configuration

In the `URL` parameter, specify the GitLab URL in the `https://gitlab.com` format, without the `/api/v4` part.

### Parameters

There are no configurable parameters.

## GitlabProjects

A datasource of the `GitlabProjects` type returns a list of projects in GitLab.

### Authorization

Authorization configuration is described in the ["GitLab external service"](../external-services/#gitlab) section.

### Response specification

Depending on the [parameter configuration](#gitlabprojectsparameters), the portal sends a GET request to the GitLab API and returns the corresponding specification.

If the `all` parameter is set to `true`, a GET request is sent to the following URL: `/api/v4/projects`.
The portal returns all available values.
[Response specification](https://docs.gitlab.com/api/projects/#list-all-projects).

If the `all` parameter is set to `false`, a GET request is sent to the following URL: `/api/v4/groups/:id/projects`.
The portal returns all available values.
[Response specification](https://docs.gitlab.com/api/projects/#list-all-projects).

If the `tags` parameter is set to `true`, the portal also retrieves git tags.
To retrieve git tags, a GET request is sent to the following URL: `/api/v4/projects/:id/repository/tags`.
The portal retrieves the list of all git tags and extends the [response specification](https://docs.gitlab.com/api/projects/#list-all-projects) with the `ddp_repository_tags` field, which matches the [list-project-repository-tags response specification](https://docs.gitlab.com/api/tags/#list-project-repository-tags).

### Configuration

In the `URL` parameter, specify the GitLab URL in the `https://gitlab.com` format, without the `/api/v4` part.

<a id="gitlabprojectsparameters"></a>

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values | Default value |
| --- | --- | --- | --- | --- |
| `all` | No | Explicitly specifies that repositories of all groups you have access to must be collected | `true`, `false` | `false` |
| `group_ids` | Yes, if `all` is `false` | The datasource collects projects of the groups with the specified IDs. Group IDs are comma-separated | Example: 1001,1002 | — |
| `tags` | No | The portal also retrieves the list of all git tags and extends the [response specification](https://docs.gitlab.com/api/projects/#list-all-projects) with the `ddp_repository_tags` field, which matches the [list-project-repository-tags response specification](https://docs.gitlab.com/api/tags/#list-project-repository-tags) | `true`, `false` | `false` |
| `include_subgroups` | No | If the `group_ids` parameter is set, the `include_subgroups` parameter defines whether to collect projects of subgroups of the specified groups | `true`, `false` | `false` |
| `tags_order_by` | No | Field used to sort tags. For the parameter description, see the [GitLab Tags API](https://docs.gitlab.com/api/tags/#list-project-repository-tags) | `updated`, `name`, `version` | `updated` |
| `tags_sort` | No | Tag sort direction. For the parameter description, see the [GitLab Tags API](https://docs.gitlab.com/api/tags/#list-project-repository-tags) | `asc`, `desc` | `desc` |
| `tags_search` | No | String used to search tags by name. For the parameter description, see the [GitLab Tags API](https://docs.gitlab.com/api/tags/#list-project-repository-tags) | String | — |

## GitlabRepositoryActivity

A datasource of the `GitlabRepositoryActivity` type returns GitLab project activity: the number of open merge requests, the latest commit, and the latest pipeline of the default branch.

The datasource populates the metrics of the microservice card on the Home page and is used by the "GitLab: repository activity" dataset.

### Authorization

Authorization configuration is described in the ["GitLab external service"](../external-services/#gitlab) section.

### Response specification

The portal sends POST requests to the GitLab GraphQL API at the `/api/graphql` URL, 50 projects per request.
For each group specified by a numeric ID, a GET request is also sent to the `/api/v4/groups/:id` URL.

For each project, the datasource returns the following fields:

| Name | Description |
| --- | --- |
| `path` | Project path without the group path |
| `gitlab_repository_id` | Numeric project ID |
| `default_branch` | Default branch |
| `open_mr_count` | Number of open merge requests |
| `last_commit_sha` | Short SHA of the latest commit in the default branch |
| `last_commit_message` | Title of the latest commit |
| `last_commit_author` | Author of the latest commit |
| `last_pipeline_id` | Number (`iid`) of the latest pipeline of the default branch |
| `last_pipeline_status` | Status of the latest pipeline in lowercase |
| `last_pipeline_web_url` | Link to the latest pipeline |

If a project has no commits or pipelines, the commit and pipeline fields are omitted.

### Configuration

In the `URL` parameter, specify the GitLab URL in the `https://gitlab.com` format, without the `/api/graphql` part.

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values | Default value |
| --- | --- | --- | --- | --- |
| `all` | No | Collect activity of all projects the token user is a member of. The `group_ids` parameter is ignored in this case | `true`, `false` | `false` |
| `group_ids` | Yes, if `all` is `false` | Comma-separated group IDs or full group paths | Example: `1001,platform/backend` | — |
| `include_subgroups` | No | Collect projects of subgroups of the specified groups | `true`, `false` | `false` |

If a group is not found or is not accessible, synchronization fails with the `gitlab group "<id>" not found or not accessible` error.

## GitlabRepositoryFiles

A datasource of the `GitlabRepositoryFiles` type reads structured files (YAML or JSON) from GitLab project repositories and returns one record for each document in a file or for each list item in a document.

### Authorization

Authorization configuration is described in the ["GitLab external service"](../external-services/#gitlab) section.

### Response specification

The portal gets the list of projects with a GET request to the GitLab API at the `/api/v4/projects` URL if the `all` parameter is set to `true`, or at the `/api/v4/groups/:id/projects` URL for each group from the `group_ids` parameter.
[Response specification](https://docs.gitlab.com/api/projects/#list-projects).

Then, for each project, the portal determines which files to read.
If the `file_path` parameter is set, only the specified file is read.
If the `file_pattern` parameter is set, the portal gets the repository tree with a GET request to the `/api/v4/projects/:id/repository/tree` URL and selects the files that match the pattern.

The portal gets the contents of each file with a GET request to the `/api/v4/projects/:id/repository/files/:file_path` URL.
A file can contain several YAML documents separated by `---`.
Each document must be an object.

Each record contains the document fields (with `document_mode: file`) or the list item fields (with `document_mode: list`), as well as the following service fields:

| Name | Description |
| --- | --- |
| `_file_path` | File path in the repository |
| `_file_name` | File name |
| `_dir` | File directory in the repository |
| `_ref` | Branch or tag the file was read from |
| `_project_id` | Numeric project ID |
| `_project_path` | Full project path (`path_with_namespace`) |
| `_project_name` | Project name |
| `_default_branch` | Default branch of the project |
| `_project_url` | Link to the project. Omitted if GitLab did not return a link to the project |
| `_web_url` | Link to the file in the GitLab web interface. Omitted together with `_project_url` |
| `_document` | The entire document the record was obtained from |
| `_index` | Index of the item in the list, starting from `0`. Only with `document_mode: list` |

Projects whose files could not be read are skipped.
A file with a syntax error is skipped entirely.
Information about skipped projects and files is written to the synchronization log.

### Configuration

In the `URL` parameter, specify the GitLab URL in the `https://gitlab.com` format, without the `/api/v4` part.

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values | Default value |
| --- | --- | --- | --- | --- |
| `all` | No | Read files from projects of all groups you have access to. The `group_ids` parameter is ignored in this case | `true`, `false` | `false` |
| `group_ids` | Yes, if `all` is `false` | Comma-separated group IDs | Example: `1001,1002` | — |
| `include_subgroups` | No | If the `group_ids` parameter is set, defines whether to read files from projects of subgroups of the specified groups | `true`, `false` | `false` |
| `topic` | No | Read files only from projects with the specified GitLab topic. Filtering is performed on the GitLab side | String | — |
| `project_pattern` | No | Regular expression for the full project path (`path_with_namespace`). Projects whose path does not match the expression are skipped | Regular expression | — |
| `max_projects` | No | Maximum number of projects to read files from. The remaining projects are not processed, and a warning is written to the synchronization log. `0` means no limit | Non-negative integer | `0` |
| `ref` | No | Branch or tag to read files from. If not set, the default branch of each project is used | String | — |
| `file_path` | Yes, if `file_pattern` is not set | Exact file path in the repository. The repository tree is not requested in this case | Example: `catalog/service.yaml` | — |
| `path` | No | Repository directory where the search for files matching the `file_pattern` pattern starts. An empty value means the repository root | Example: `catalog` | `""` |
| `file_pattern` | Yes, if `file_path` is not set | File name pattern in the [path.Match](https://pkg.go.dev/path#Match) format. If the pattern contains `/`, it is matched against the full file path in the repository | Example: `*.yaml` | — |
| `recursive` | No | Search for files matching the `file_pattern` pattern in nested directories | `true`, `false` | `true` |
| `document_mode` | No | How a document is converted into records: `file` — the document becomes a single record; `list` — each list item under the `list_path` key becomes a record | `file`, `list` | `file` |
| `list_path` | Yes, if `document_mode` is `list` | Top-level document key that holds the list of items | Example: `items` | — |

Exactly one of the `file_path` and `file_pattern` parameters must be set.
If the parameters are set incorrectly, synchronization fails.

### Example

Get records from the `services` list in the `catalog/services.yaml` file of projects in the `1001` group with the `ddp` topic:

```yaml
group_ids: "1001"
topic: ddp
file_path: catalog/services.yaml
document_mode: list
list_path: services
```

## HarborArtifacts

A datasource of the `HarborArtifacts` type collects information about all artifacts in Harbor.

### Authorization

Authorization configuration is described in the ["Harbor external service"](../external-services/#harbor) section.

### Response specification

The portal gets the list of all projects and the repositories they contain, then sends a GET request to the Harbor API for each available project: `/api/v2.0/projects/{project_name}/repositories/{repository_name}/artifacts`.
The response specification is available [in the Harbor interface](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/).

### Configuration

In the `URL` parameter, specify the Harbor URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## HarborProjects

A datasource of the `HarborProjects` type collects information about all projects in Harbor.

### Authorization

Authorization configuration is described in the ["Harbor external service"](../external-services/#harbor) section.

### Response specification

The portal gets the list of all available projects by sending GET requests to the Harbor API: `/api/v2.0/projects`.
The response specification is available [in the Harbor interface](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/).

### Configuration

In the `URL` parameter, specify the Harbor URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## HarborRepositories

A datasource of the `HarborRepositories` type collects information about all repositories in Harbor.

### Authorization

Authorization configuration is described in the ["Harbor external service"](../external-services/#harbor) section.

### Response specification

The portal gets the list of all projects, then gets the list of all repositories in each project by sending GET requests to the Harbor API: `/api/v2.0/projects/{project_name}/repositories`.
The response specification is available [in the Harbor interface](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/).

### Configuration

In the `URL` parameter, specify the Harbor URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## HarborTags

A datasource of the `HarborTags` type collects information about all artifact tags in Harbor.

### Authorization

Authorization configuration is described in the ["Harbor external service"](../external-services/#harbor) section.

### Response specification

The portal gets the list of all projects and the repositories they contain, then sends a GET request to the Harbor API for each available project: `/api/v2.0/projects/{project_name}/repositories/{repository_name}/artifacts`.
Then all tags of all artifacts (the `tags` field) are collected, and the result is returned as an array.
The response specification is available [in the Harbor interface](https://goharbor.io/docs/main/working-with-projects/using-api-explorer/).

### Configuration

In the `URL` parameter, specify the Harbor URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## HelmReleases

A datasource of the `HelmReleases` type returns a list of all HelmReleases in a Kubernetes cluster.

### Authorization

Authorization configuration is described in the ["Kubernetes external service"](../external-services/#kubernetes) section.

Authentication in Kubernetes is described [in the official documentation](https://kubernetes.io/docs/reference/access-authn-authz/authentication/).

### Response specification

The portal returns all HelmReleases in the Kubernetes cluster. Specification:

```json
[
  {
    "name": "string",              // Name.
    "info": {
      "first_deployed": "string",  // Date and time of the first deployment.
      "last_deployed": "string",   // Date and time of the last deployment.
      "deleted": "string",         // Date and time of deletion (can be an empty string).
      "description": "string",     // Description.
      "status": "string",          // Status.
      "notes": "string"            // Notes.
    },
    "chart": {
      "metadata": {
        "name": "string",          // Chart name.
        "home": "string",          // Home page.
        "sources": "array",        // Array of strings with source URLs.
        "version": "string",       // Chart version.
        "description": "string",   // Chart description.
        "keywords": "array",       // Array of strings with keywords.
        "maintainers": "array",    // Array of objects with maintainer information.
        "icon": "string",          // Icon URL.
        "apiVersion": "string",    // API version.
        "appVersion": "string"     // Application version.
      },
      "templates": [               // Array of template objects.
        {
          "name": "templates/NOTES.txt",
          "data": "..."
        },
        {
          "name": "templates/_helpers.tpl",
          "data": "..."
        }
        // ... other templates.
      ],        
      "values": "object",          // Object with the available Helm chart settings.
      "schema": "null",            // Schema (can be null).
      "files": [                   // Array of file objects.
        {
          "name": ".helmignore",
          "data": "..."
        },
        {
          "name": "LICENSE",
          "data": "..."
        }
        // ... other files.
      ]             
    },
    "config": {        // Current configuration.
      "caSecretName": "string",
      "cache": {
        "enabled": "boolean", 
        "expireHours": "integer"
      }
      // ... other settings.
    },
    "manifest": "string", // Rendered manifests.
    "version": "integer", // HelmRelease version.
    "namespace": "string" // Namespace where the release is deployed.
  },
  // ... other HelmRelease resources.
]

```

### Configuration

In the `URL` parameter, specify the Kubernetes API URL in the `https://api.example.com` format.

### Parameters

There are no configurable parameters.

## KafkaAcls

A datasource of the `KafkaAcls` type collects information about the available ACLs.

### Authorization

The portal supports authentication in Kafka using [SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain), [SASL/SCRAM](https://kafka.apache.org/documentation/#security_sasl_scram).

### Response specification

The portal requests information about the ACLs configured in Kafka. The retrieved data is provided in the following format:

```json
[
  {
    "Cluster": "string",             // Kafka cluster name.
    "ResourceType": "string",        // Resource type (TOPIC, GROUP, and others).
    "ResourceName": "string",        // Resource name.
    "PatternType": "string",         // Pattern type (LITERAL, PREFIXED, and others).
    "Principal": "string",           // User principal (for example, "User:Alice").
    "Host": "string",                // Host (usually "*" for any host).
    "Operation": "string",           // Operation (READ, WRITE, DESCRIBE, and others).
    "PermissionType": "string"       // Permission type (ALLOW, DENY).
  },
  // ... other ACL entries.
]
```

### Configuration

In the `URL` parameter, specify the Kafka URL in the `example.com` format.

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values |
| --- | --- | --- | --- |
| SecurityProtocol | Yes | Protocol used to connect to Kafka. See the [Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
| SaslMechanism | No | Authentication mechanism used by SASL. Required with the SASL_PLAINTEXT or SASL_SSL protocol. See the [Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
| User | Yes | Username for connecting to Kafka | — |
| Pass | Yes | User password for connecting to Kafka | — |

## KafkaBrokers

A datasource of the `KafkaBrokers` type collects information about the available brokers.

### Authorization

The portal supports authentication in Kafka using [SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain), [SASL/SCRAM](https://kafka.apache.org/documentation/#security_sasl_scram).

### Response specification

The portal sends several requests to Kafka to retrieve information about the available brokers.
The retrieved data is provided in the following format:

```json
{
  "Cluster": "string",          // Kafka cluster name (from kafkaBrokerMetadata.Cluster).
  "Leader": "boolean",          // Whether the broker is the controller leader (true/false).
  "NodeID": "number",           // Unique broker ID (from broker.NodeID).
  "Port": "number",             // Broker port (from broker.Port).
  "Host": "string",             // Broker host (from broker.Host).
  "Rack": "string|null",        // Broker rack (availability zone) (from broker.Rack).
  "Configs": {                  // Broker configurations.
    {
      "Name": "number",         // Unique broker ID.
      "Configs": [              // Array of individual configurations.
        {
          "Key": "listeners",       // Configuration key.
          "Value": "CLIENT://:9092,INTERNAL://:9094", // Configuration value.
          "Source": "STATIC_BROKER_CONFIG" // Configuration source.
        },
        {
          "Key": "log.retention.hours", // Configuration key.
          "Value": 168,                 // Configuration value.
          "Source": "DEFAULT_CONFIG"    // Configuration source.
        },
        {
          "...": "..."                  // Other parameters.
          // For the full list of parameters, see the official documentation:
          // https://kafka.apache.org/documentation/#brokerconfigs.
        }
      ]
    }
  }
}
```

### Configuration

In the `URL` parameter, specify the Kafka URL in the `example.com` format.

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values |
| --- | --- | --- | --- |
| SecurityProtocol | Yes | Protocol used to connect to Kafka. See the [Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
| SaslMechanism | No | Authentication mechanism used by SASL. Required with the SASL_PLAINTEXT or SASL_SSL protocol. See the [Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
| User | Yes | Username for connecting to Kafka | — |
| Pass | Yes | User password for connecting to Kafka | — |

## KafkaTopics

A datasource of the `KafkaTopics` type collects information about the available topics in Kafka.

### Authorization

The portal supports authentication in Kafka using [SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain), [SASL/SCRAM](https://kafka.apache.org/documentation/#security_sasl_scram).

### Response specification

The portal sends several requests to Kafka to retrieve information about the available topics.
The retrieved data is provided in the following format:

```json
{
  "Cluster": "string",           // Kafka cluster name.
  "Topic": "string",             // Topic name.
  "ID": "string",                // Unique topic identifier.
  "IsInternal": "boolean",       // Whether the topic is internal (example of an internal topic: __consumer_offsets).
  "Partitions": "number",        // Number of partitions in the topic.
  "Configs": {                   // Topic configurations (dynamic parameters).
    "retention.ms": 604800000,   // Example parameter: message retention time.
    "cleanup.policy": "delete",  // Example parameter: cleanup policy.
    "...": "..."                 // Other parameters.
    // For the full list of parameters, see the official documentation:
    // https://kafka.apache.org/documentation/#topicconfigs.
  }
}
```

### Configuration

In the `URL` parameter, specify the Kafka URL in the `example.com` format.

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values |
| --- | --- | --- | --- |
| SecurityProtocol | Yes | Protocol used to connect to Kafka. See the [Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol) | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL |
| SaslMechanism | No | Authentication mechanism used by SASL. Required with the SASL_PLAINTEXT or SASL_SSL protocol. See the [Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512 |
| User | Yes | Username for connecting to Kafka | — |
| Pass | Yes | User password for connecting to Kafka | — |

## KubernetesResources

A datasource of the `KubernetesResources` type returns a list of Kubernetes resources.
The type of returned resources is set in the datasource parameters.
Both built-in Kubernetes resource types and any custom resources are supported.

### Authorization

Authorization configuration is described in the ["Kubernetes external service"](../external-services/#kubernetes) section.

### Response specification

The response specification depends on the type of resource you want to retrieve.
For exact information, refer to the documentation.
For built-in Kubernetes resources, the [official documentation](https://kubernetes.io/docs/reference/kubernetes-api/) is available.

If you cannot access the documentation, you can get the specification description using the `d8 k explain` command.

Example for a Deployment:

`d8 k explain deployment --recursive`

Sample output:

```console
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
    
FIELDS:
  apiVersion <string>
  kind <string>
  metadata <ObjectMeta>
    annotations <map[string]string>
    creationTimestamp <string>
...
```

The specification description is located under `FIELDS:`.

### Configuration

In the `URL` parameter, specify the Kubernetes API URL in the `https://api.example.com` format.

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values |
| --- | --- | --- | --- |
| `apiGroup` | No | API group of the resource. Leave the field empty for resources in the core API group | [See how to determine the required group and version](#determining-the-required-group-and-version) |
| `version` | Yes | API version of the resource | [See how to determine the required group and version](#determining-the-required-group-and-version) |
| `isNamespaced` | Yes | Whether the resource is namespaced. You can check this using the `d8 k api-resources` command | `true`, `false` |
| `namespace` | No | Namespace to collect resources from. If not specified, resources are collected from all namespaces. The value is taken into account only if `isNamespaced` is `true` | — |
| `resource` | Yes | Resource name. Specified in lowercase plural form, as in the NAME field of the `d8 k api-resources` command output | — |

### Examples

Collect all Ingress resources of a Kubernetes cluster:

```yaml
apiGroup: networking.k8s.io
version: v1
isNamespaced: true
resource: ingresses
```

Collect all pods of a Kubernetes cluster:

```yaml
version: v1
isNamespaced: true
resource: pods
```

Collect all pods of a Kubernetes cluster in the `d8-development-platform` namespace:

```yaml
version: v1
isNamespaced: true
resource: pods
namespace: d8-development-platform
```

Collect all namespaces of a Kubernetes cluster:

```yaml
version: v1
isNamespaced: false
resource: namespaces
```

Collect all ModuleRelease custom resources of a Kubernetes cluster:

```yaml
apiGroup: deckhouse.io
version: v1alpha1
isNamespaced: false
resource: modulereleases
```

### Determining the required group and version

Each resource type has its own version and group.
For the full list of API resources with their groups and versions, see the [Kubernetes documentation](https://kubernetes.io/docs/reference/kubernetes-api/).

If you do not know which group and version are required, you can look up the current values.
There are several ways to do this, for example, using the `d8 k` utility.
The `d8 k explain` command shows the `version` and `apiGroup` of a resource.

Example:

```bash
d8 k explain deployment
```

Output:

```console
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
    
FIELDS:
...
```

## NexusArtifacts

A datasource of the `NexusArtifacts` type returns a list of artifacts in Nexus.

### Authorization

Authorization configuration is described in the ["Nexus external service"](../external-services/#nexus) section.

### Response specification

The portal builds a list of available repositories, then sends a GET request to the Nexus API for each repository: `/service/rest/v1/components`.
[Response specification](https://help.sonatype.com/en/components-api.html).

### Configuration

In the `URL` parameter, specify the Nexus URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## NexusRepositories

A datasource of the `NexusRepositories` type returns a list of repositories in Nexus.

### Authorization

Authorization configuration is described in the ["Nexus external service"](../external-services/#nexus) section.

### Response specification

The portal sends a GET request to the Nexus API: `/service/rest/v1/repositories`.
[Response specification](https://help.sonatype.com/en/repositories-api.html).

### Configuration

In the `URL` parameter, specify the Nexus URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## PrometheusMetrics

A datasource of the `PrometheusMetrics` type returns the result of a PromQL query in Prometheus.

### Authorization

Authorization configuration is described in the ["Prometheus external service"](../external-services/#prometheus) section.

### Response specification

The portal sends a GET request to the Prometheus API: `/api/v1/query`.
[Response specification](https://prometheus.io/docs/prometheus/latest/querying/api/#instant-queries).

### Configuration

In the `URL` parameter, specify the Prometheus API URL in the `https://example.com/api/v1/query` format.
In the "Query" parameter, specify a [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) query used to build the response.

### Parameters

There are no configurable parameters.

## SonarqubeProjects

A datasource of the `SonarqubeProjects` type returns a list of projects in SonarQube.

### Authorization

Authorization configuration is described in the ["SonarQube external service"](../external-services/#sonarqube) section.

### Response specification

The portal sends a GET request to the SonarQube API: `/api/projects/search`.
The system collects all available `components` and returns them as an array.
[`components` specification](https://next.sonarqube.com/sonarqube/web_api/api/projects/search).

### Configuration

In the `URL` parameter, specify the SonarQube URL in the `https://example.com` format.

### Parameters

There are no configurable parameters.

## GenericAPI

A datasource of the `GenericAPI` type lets you connect to any REST API and retrieve data in JSON format.
Various pagination types and configurable request parameters are supported.

### Authorization

GenericAPI supports any authentication type through HTTP header configuration. The most common methods are:

Bearer Token:

```text
Authorization: Bearer <TOKEN>
```

Basic Authentication:

```text
Authorization: Basic <BASE64_CREDENTIALS>
```

API Key:

```text
X-API-Key: <API_KEY>
```

### Configuration

When configuring the datasource, set the following parameters:

- `URL`: Base API URL in the `https://api.example.com` format.
- `Method`: HTTP method (GET, POST, PUT, PATCH, DELETE).
- `Query`: Additional query parameters (for example, for filtering or searching).

### Parameters

The datasource supports the following parameters:

| Name | Required | Description | Possible values | Example | Default value |
| --- | --- | --- | --- | --- | --- |
| `paginationType` | No | Pagination type for processing large amounts of data | `none`, `offset`, `cursor`, `page`, `link_header` | `none` | `none` |
| `path` | No | API endpoint path appended to the base URL | Any string starting with `/` | `/api/v1/users`, `/projects` | `""` |
| `dataPath` | No | Path to the data in the response (a chain of dot-separated keys) | Any string | `data`, `results.items`, `.` | `.` |
| `responseFormat` | No | Response format | `array`, `map` | `array`, `map` | `array` |
| `mapKeyField` | No | With `responseFormat` set to `map`: name of the field to write the JSON object key to | Any string | `slug`, `externalId` | `""` |
| `pageParam` | No | Name of the page number parameter (for page pagination) | Any string | `page`, `_page`, `pageNumber` | `page` |
| `offsetParam` | No | Name of the offset parameter (for offset pagination) | Any string | `offset`, `skip` | `offset` |
| `limitParam` | No | Name of the parameter for the number of items per page (for offset pagination) | Any string | `limit`, `_limit` | `limit` |
| `sizeParam` | No | Name of the page size parameter (for page pagination) | Any string | `size`, `pageSize`, `_size` | `size` |
| `cursorParam` | No | Name of the cursor parameter (for cursor pagination) | Any string | `cursor`, `after`, `next` | `cursor` |
| `pageSize` | No | Number of items per page for pagination | Positive integer | `10`, `20`, `50`, `100` | `100` |
| `maxPages` | No | Maximum number of pages to load for all pagination types. `0` means no limit | Non-negative integer | `10`, `50` | `0` |
| `nextCursorPath` | Yes, with `paginationType: cursor` | Name of the field in the last item of the received data that contains the next page cursor | Any string | `next_cursor`, `cursor` | `""` |
| `requestBody` | No | Request body for the POST/PUT/PATCH methods | Any string | `{"query": "example"}` | `""` |

### Response specification

The expected response format is set by the `responseFormat` parameter (`array` by default).

Example response for `responseFormat: array`:

```json
[
  {
    "id": 1,
    "name": "Example Item",
    "description": "Description of the item",
    "created_at": "2023-01-01T00:00:00Z"
  },
  {
    "id": 2,
    "name": "Another Item",
    "description": "Another description",
    "created_at": "2023-01-02T00:00:00Z"
  }
]
```

Example response for `responseFormat: map`:

```json
{
  "billing": { "title": "Billing", "owner": "team-a" },
  "shipping": { "title": "Shipping", "owner": "team-b" }
}
```

Before processing, a response with `responseFormat: map` is converted into an array based on the `mapKeyField` parameter value.

With `mapKeyField: example`, an array is passed for processing with an added `example` field whose value is taken from the object key:

```json
[
  { "example": "billing", "title": "Billing", "owner": "team-a" },
  { "example": "shipping", "title": "Shipping", "owner": "team-b" }
]
```

If `mapKeyField` is not set (an empty string), an array is passed for processing without adding a new field:

```json
[
  { "title": "Billing", "owner": "team-a" },
  { "title": "Shipping", "owner": "team-b" }
]
```

### Pagination types

The following pagination types are supported:

- `none`: No pagination; all data is retrieved in a single request.

- `offset`: Pagination by offset and limit:
  - Uses the `offsetParam` and `limitParam` parameters.
  - The offset starts at `0` and increases by the number of received records.
  - Example: `?offset=0&limit=100`, then `?offset=100&limit=100`.

- `page`: Pagination by page number and size:
  - Uses the `pageParam` and `sizeParam` parameters.
  - Example: `?page=1&size=20`.

- `cursor`: Cursor-based pagination:
  - Uses the `cursorParam` and `nextCursorPath` parameters.
    The `nextCursorPath` parameter is required. If it is not set, only the first page is loaded.
  - The next page cursor is taken from the `nextCursorPath` field of the last item of the received data and passed in the `cursorParam` query parameter.
    Loading stops when the cursor field is missing, empty, or not a string.
  - Example: `?cursor=eyJpZCI6MTIzfQ==`.

- `link_header`: Pagination using the Link header:
  - Uses the `Link` header in the response to determine the next page.
  - RFC 5988 standard.

### Configuration examples

JSONPlaceholder API (no pagination):

```yaml
url: https://jsonplaceholder.typicode.com
path: /users
paginationType: none
dataPath: .
```

GitLab API (page pagination):

```yaml
url: https://gitlab.example.com
path: /api/v4/projects
paginationType: page
pageParam: page
sizeParam: per_page
pageSize: 20
headers:
  Authorization: Bearer <TOKEN>
```

DefectDojo API (offset pagination):

```yaml
url: https://defectdojo.example.com
path: /api/v2/engagements
paginationType: offset
limitParam: limit
pageSize: 100
dataPath: results
query: "product=42"
headers:
  Authorization: Token <TOKEN>
```

GitHub API (Link header pagination):

```yaml
url: https://api.github.com
path: /repos/owner/repo/issues
paginationType: link_header
headers:
  Authorization: Bearer <TOKEN>
  Accept: application/vnd.github.v3+json
```

API with nested data:

```yaml
url: https://api.example.com
path: /data
dataPath: response.data.items
paginationType: page
pageParam: page
sizeParam: limit
query: "filter=active&sort=name"
```

POST request with a body:

```yaml
url: https://api.example.com
path: /search
method: POST
requestBody: '{"query": "example", "filters": {"status": "active"}}'
query: "include=metadata&format=json"
headers:
  Content-Type: application/json
  Authorization: Bearer <TOKEN>
```
