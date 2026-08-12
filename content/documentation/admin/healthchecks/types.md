---
title: Health check types
---

## Property

A `Property` rule checks whether a specific entity parameter matches a template expression.

The rule configuration contains one parameter: an expression written in [Go template syntax](https://developer.hashicorp.com/nomad/docs/reference/go-template-syntax).

Expression examples:

- `{{ eq .entity.properties.lifecycle "deployed" }}` — the `lifecycle` property must equal `"deployed"`;
- `{{ lt .entity.properties.vulnerabilities 10 }}` — the `vulnerabilities` property must be less than `10`.

## Prometheus

A Prometheus rule checks whether a specified metric meets a configured threshold. The configuration contains a PromQL query that must return a Scalar or a single-value Vector.

Templating is supported. For example:

```go
avg(ingress_nginx_detail_request_seconds_sum{location="/{{ .entity.slug }}"})
```

In this example, the entity identifier replaces `{{ .entity.slug }}`.

### Configuration parameters

| Name      | Description                                                | Allowed values                          |
|-----------|------------------------------------------------------------|-----------------------------------------|
| Query     | PromQL query for a Prometheus metric                       |                                         |
| Operator  | Comparison operator applied to the query result and threshold | Equal, NotEqual, LessThan, GreaterThan |
| Threshold | Threshold compared with the query result                   |                                         |

{{< alert level="info" >}}
Each entity check sends a separate request to Prometheus. Account for this when planning system load.
{{< /alert >}}

### Authorization

Authorization is configured in the [Prometheus external service](../external-services/#prometheus) section.

## GitLab Pipeline

A `GitlabPipeline` rule checks whether the latest GitLab pipeline for the selected Ref (branch or tag) has the configured status.

All text fields support templating. For example, specify the following expression in the `Ref` field:

```go
{{ .entity.properties.mainBranch }}
```

During the check, the value of the checked entity's `mainBranch` parameter replaces the expression.

### Configuration parameters

| Name       | Description                                                  | Allowed values                                                                                                      |
|------------|--------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| Project ID | GitLab project ID                                            |                                                                                                                     |
| Ref        | Branch or tag whose latest pipeline is checked               |                                                                                                                     |
| Status     | Pipeline status considered successful                        | created, waiting_for_resource, preparing, pending, running, success, failed, canceled, skipped, manual, scheduled |

{{< alert level="info" >}}
Each entity check sends a separate request to GitLab. Account for this when planning system load.
{{< /alert >}}

### Authorization

Authorization is configured in the [GitLab external service](../external-services/#gitlab) section.

## DefectDojo Findings

A `DefectDojoFindings` rule checks the number of active vulnerabilities at each severity level for a specified DefectDojo product.

The check uses the conditions in the `conditions` block. For each severity, you can specify the expected number of vulnerabilities and a comparison operator, for example, Critical < 5.

All text fields support templating. For example, you can insert the product name from the entity parameters:

```go
{{ .entity.properties.defectdojo_product_key }}
```

### Configuration parameters

| Name         | Description                                                   | Allowed values |
|--------------|---------------------------------------------------------------|----------------|
| Product name | Product identifier in DefectDojo                             |                |
| Conditions   | Conditions for comparing vulnerability counts by severity    |                |

#### Conditions

Each condition is an object in the following form:

```yaml
conditions:
  - severity: Critical
    operator: "<"
    value: "5"
  - severity: High
    operator: "<="
    value: "10"
```

| Field    | Description                       | Allowed values                            |
|----------|-----------------------------------|-------------------------------------------|
| Severity | Severity level                    | Total, Critical, High, Medium, Low, Info  |
| Operator | Comparison operator               | ==, !=, <, <=, >, >=                      |
| Value    | Target value for the comparison   |                                           |

### Authorization

Authorization is configured in the [DefectDojo external service](../external-services/#defectdojo) section.

## CodeScoring Vulnerabilities

A `CodeScoringVulnerabilities` rule checks the number of vulnerabilities at each severity level for a specified CodeScoring project.

The check uses the conditions in the `conditions` block. For each severity, you can specify the expected number of vulnerabilities and a comparison operator. Both CVSS2 and CVSS3 metrics are supported.

All text fields support templating. For example, you can insert the project ID from the entity parameters:

```go
{{ .entity.properties.codescoring_project_id }}
```

### Configuration parameters

| Name       | Description                                                   | Allowed values |
|------------|---------------------------------------------------------------|----------------|
| Project ID | Project identifier in CodeScoring                             |                |
| Conditions | Conditions for comparing vulnerability counts by severity    |                |

#### Conditions

Each condition is an object in the following format:

```yaml
conditions:
  - severity: CRITICAL
    operator: "<"
    value: "5"
    cvss: "cvss3"
  - severity: HIGH
    operator: "<="
    value: "10"
    cvss: "cvss2"
```

| Field    | Description                     | Allowed values                                                                          |
|----------|---------------------------------|-----------------------------------------------------------------------------------------|
| Severity | Severity level                  | CVSS3: CRITICAL, HIGH, MEDIUM, LOW, NONE, UNKNOWN. CVSS2: HIGH, MEDIUM, LOW, NONE       |
| Operator | Comparison operator             | ==, !=, <, <=, >, >=                                                                    |
| Value    | Target value for the comparison |                                                                                         |
| CVSS     | CVSS metric version             | cvss2, cvss3                                                                            |

### Authorization

Authorization is configured in the [CodeScoring external service](../external-services/#codescoring) section.

## SonarQube Metrics

A `SonarqubeMetrics` rule checks SonarQube project metrics against configured conditions.

The check calls the SonarQube REST API endpoint `/api/measures/component` and compares current metric values with the expected values specified under **Conditions**.

All text fields support templating. For example, you can insert the component key from the entity parameters:

```go
{{ .entity.properties.sonarqube_project_key }}
```

### Configuration parameters

| Name        | Required | Description                              | Allowed values |
|-------------|----------|------------------------------------------|----------------|
| Project key | Yes      | Project identifier in SonarQube          |                |
| Branch      | No       | Project branch from which metrics are read |              |
| Conditions  | Yes      | Conditions for comparing SonarQube metrics |              |

#### Conditions

Each condition is an object in the following form:

```yaml
conditions:
  - metric: coverage
    operator: "<"
    value: "5"
  - metric: bugs
    operator: "<="
    value: "10"
```

| Field    | Description                                          | Allowed values                                      |
|----------|------------------------------------------------------|-----------------------------------------------------|
| Metric   | SonarQube metric specified by its metric key         | See the metric list in the official SonarQube documentation |
| Operator | Comparison operator                                  | ==, !=, <, <=, >, >=                                |
| Value    | Target value for the comparison                      |                                                     |

See the [list of available metrics](https://docs.sonarsource.com/sonarqube-server/latest/user-guide/code-metrics/metrics-definition) for the current SonarQube version.

### Authorization

Authorization is configured in the [SonarQube external service](../external-services/#sonarqube) section.

## SonarQube Quality Gate

A `SonarqubeQualityGate` rule checks the Quality Gate status of a SonarQube project. The check calls the SonarQube REST API endpoint `/api/qualitygates/project_status`.

The rule returns `true` (passes) if the project Quality Gate has the `OK` status. It returns `false` (fails) for all other statuses: `WARN`, `ERROR`, or `NONE`.

All text fields support templating. For example, you can insert the project key from the entity parameters:

```go
{{ .entity.properties.sonarqube_project_key }}
```

### Configuration parameters

| Name        | Required | Description                                                                                  | Allowed values |
|-------------|----------|----------------------------------------------------------------------------------------------|----------------|
| Project key | Yes      | Project identifier in SonarQube                                                              |                |
| Branch      | No       | Project branch whose Quality Gate is checked. If omitted, the main branch is checked         |                |

{{< alert level="info" >}}
Each entity check sends a separate request to SonarQube. Account for this when planning system load.
{{< /alert >}}

### Authorization

Authorization is configured in the [SonarQube external service](../external-services/#sonarqube) section.

## URL

A `URL` rule checks the availability of an HTTP or HTTPS endpoint. It sends a request with the configured parameters and evaluates the result against one or more Go template conditions. Conditions can check the response code, headers, response body, and entity parameters.

All string fields (URL, request body, and headers) support templating with entity data (`.entity`), for example, `{{ .entity.slug }}`.

### Configuration parameters

| Name                | Required | Description                                                                                                                                                        | Examples              |
|---------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| URL                 | Yes      | Full request URL                                                                                                                                                   | `https://example.com` |
| Method              | No       | HTTP request method                                                                                                                                                | GET, POST             |
| Query               | No       | Query value added to the outgoing request                                                                                                                          |                       |
| Save details        | No       | If enabled, the check table stores and displays the response body and status (code and headers). If disabled, it shows only the condition, result, and error       |                       |
| Conditions          | No       | Go template expressions used to evaluate the response                                                                                                              | See below             |
| Multiple conditions | No       | `AllOf` requires all conditions to pass; `AnyOf` requires at least one                                                                                             | AllOf, AnyOf          |
| Request body        | No       | Request body in YAML format. After template substitution, it is converted to JSON and sent                                                                          |                       |

If no conditions are specified, the response code is checked against `200` by default: `{{ eq .status.code 200 }}`.

### Conditions

Conditions can use:

- `.status` — response data: `.status.code` (HTTP code), `.status.status` (status string), `.status.headers` (headers), and `.status.contentLength`. `.status.headers` is a map of header names to value arrays, with names in lowercase. Example: access the first header value with `{{ index (index .status.headers "content-type") 0 }}`; iterate over values with `{{ range index .status.headers "set-cookie" }}...{{ end }}`.
- `.response` — response body with converted types (for a JSON response).
- `.entity` — parameters of the checked entity.

Condition examples:

```go
{{ eq .status.code 200 }}
{{ eq (index (index .status.headers "content-type") 0) "application/json" }}
{{ gt .status.contentLength 0 }}
```

### Request body example

Specify the **Request body** field in YAML format. For example:

```yaml
id: "{{ .entity.id }}"
name: "{{ .entity.name }}"
```

### Result evaluation

The check passes when all conditions pass in `AllOf` mode or at least one passes in `AnyOf` mode. If evaluating a condition returns an error—for example, the response is not JSON but the condition uses `.response`—the check returns an error.
