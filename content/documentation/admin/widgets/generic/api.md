---
title: API
weight: 10
---

The widget displays an API specification from a file in a GitLab repository or from a URL in OpenAPI (Swagger) or Protobuf format. For an OpenAPI specification in a YAML or JSON file, the widget displays the Swagger UI. In all other cases, it displays the specification as text.

## General configuration

| Name               | Required | Description                                      | Possible values                     | Default |
| ------------------ | -------- | ------------------------------------------------ | ----------------------------------- | ------- |
| Specification type | Yes      | Specification type                               | OpenAPI (Swagger), Protocol Buffers | -       |
| Source type        | Yes      | Source from which the specification file is loaded | URL, GitLab                       | -       |

## Source type configuration: URL

| Name    | Required | Description                                      | Default |
| ------- | -------- | ------------------------------------------------ | ------- |
| URL     | Yes      | URL of the specification file                    | -       |
| Headers | No       | Headers used to access the specification file    | -       |

## Source type configuration: GitLab

| Name        | Required | Description                                                   | Default |
| ----------- | -------- | ------------------------------------------------------------- | ------- |
| GitLab URL  | Yes      | GitLab URL                                                    | -       |
| Project ID  | Yes      | ID of the project containing the specification file           | -       |
| Branch      | Yes      | Branch containing the specification file                      | -       |
| File path   | Yes      | Path to the specification file relative to the repository root | -      |

## Authorization

Authorization is configured in [External services](../../external-services/#gitlab).
