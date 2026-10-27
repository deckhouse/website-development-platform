---
title: Creating and publishing a microservice template
weight: 40
description: Step-by-step creation of a Go service template pack for DDP — the manifest, parameters, files, validation with the ddp-template utility, publishing to a container registry, synchronization, and creating a microservice from the template.
---

This guide describes creating an HTTP service template in Go and publishing it to the portal's template registry.
As a result, the template is shown in the gallery of the "Create microservice" Home page section, and the repository created from it contains code, CI, and a Helm chart.

## Before you begin

To follow this guide, prepare the following:

- A container registry with OCI Distribution Spec v2, for example, `registry.example.com`, and a token with write permissions for the `ddp-templates/go-service` repository.
- The `ddp-template` utility.
- A connection to this registry in the portal's template registry with the `ddp-templates` repository prefix.
  Connecting a registry is described in the [Template registry](../templates/registry/#connecting-a-registry) section.

## Pack structure

Create a pack directory with the following files:

```text
go-service/
  template.yaml
  values.yaml
  .templateignore
  .editorconfig
  .gitlab-ci.yml
  go.mod
  cmd/{{ .slug }}/main.go
  deploy/helm/Chart.yaml
  deploy/helm/values.yaml
  deploy/helm/templates/deployment.yaml
```

The `cmd/{{ .slug }}` directory is named after the microservice identifier.

### Manifest

The `template.yaml` manifest sets the template identifier and the upgrade rules.
CI and the Helm chart belong to the template and are overwritten on upgrade, while `.editorconfig` is added only if it is missing from the repository:

```yaml
id: go-service
name: Go service
description: HTTP service on Go with GitLab CI and Helm chart
language: Go
tags:
  - http
  - helm
update:
  strategy: selective_additive
  overwrite:
    - .gitlab-ci.yml
    - deploy/**
  addIfMissing:
    - .editorconfig
```

The `id` value matches the name of the pack repository in the registry: `ddp-templates/go-service`.

### Parameters

The `values.yaml` file describes the template parameters.
The identifier, microservice name, and system are filled in from the run context and are not shown in the form; the user enters the port at the "Parameters" step:

```yaml
type: object
required:
  - slug
  - name
  - system
  - port
properties:
  slug:
    type: string
    title: Service identifier
    x-ddp-from: context.microservice.slug
  name:
    type: string
    title: Service name
    x-ddp-from: context.microservice.name
  system:
    type: string
    title: System
    x-ddp-from: context.system.slug
  port:
    type: integer
    title: HTTP port
    description: Port the service listens on
    default: 8080
```

### Repository files

The repository files use the template parameters. The `go.mod` file:

```go-html-template
module {{ .slug }}

go 1.23
```

The `cmd/{{ .slug }}/main.go` file:

```go-html-template
package main

import (
    "log"
    "net/http"
)

func main() {
    // Service {{ .name }} of system {{ .system }}.
    log.Fatal(http.ListenAndServe(":{{ .port }}", nil))
}
```

The `deploy/helm/values.yaml` file:

```go-html-template
name: {{ .slug }}
system: {{ .system }}
port: {{ .port }}
```

The `deploy/helm/templates/deployment.yaml` file uses the Helm `{{ .Values }}` syntax.
To prevent the portal from processing it as a template, add the path to `.templateignore`:

```text
# Helm templates are copied as is.
deploy/helm/templates/**
```

## Validating the pack

Validate the pack with the `ddp-template` utility:

```shell
ddp-template lint ./go-service
```

The command checks the manifest, the parameter schema, `.templateignore`, and the upgrade path lists, and performs a trial render.
Exit code `0` means there are no errors.

To see the render result, create a `test-values.yaml` file with test values and render the pack:

```yaml
slug: billing-api
name: Billing API
system: billing
port: 8080
```

```shell
ddp-template render ./go-service --values ./test-values.yaml --output ./render-out
```

The `./render-out` directory will contain the repository files: `cmd/billing-api/main.go`, `go.mod` with `module billing-api`, and the unchanged Helm templates.

## Publishing

Publish version `1.0.0` to the container registry:

```shell
export DDP_TEMPLATE_REGISTRY_TOKEN=<REGISTRY_TOKEN>
ddp-template publish ./go-service --version 1.0.0 \
  --registry registry.example.com \
  --repository-prefix ddp-templates
```

Where `<REGISTRY_TOKEN>` is a container registry token with write permissions.

The command outputs JSON with the `templateId`, `version`, `ociRef`, and `sha256` fields.
A published version cannot be overwritten. To make changes, publish a new version.

## Synchronizing

The portal loads the version during the next template registry sync. To load the version immediately, follow these steps:

1. Go to "Administration" → "Integrations" → "Templates" → "Sync runs".
1. Click "Sync" and wait for the "Success" status.
1. On the "Templates" tab, find `go-service` version `1.0.0` and make sure it has the "available" status and the "Latest" tag.

If the "Repository IDs" field of the connection is filled in, add `go-service` to it before syncing.

If the version has the "invalid" status, the reason is shown in the "Status reason" column.
Fix the pack and publish version `1.0.1`.

## Binding processes

Bind processes to the template in the Home page mapping rules:

1. Go to "Administration" → "Directories" → "Mapping rules" and open the default set.
1. On the "Catalog mapping" tab, expand "Microservices" → "Template actions" and click "Add template".
1. Select the `go-service` template and the action processes:
   - "Create microservice": "Create microservice from template"
   - "Deploy microservice to an environment": "Deploy microservice to an environment"
   - "Delete microservice from an environment": "Delete microservice from an environment"
   - "Upgrade microservice from template": "Upgrade microservice from template"
1. Click "Save".

## Verifying the result

Create a microservice from the template:

1. Open the Home page → "Create microservice" and select the "Go service" card.
1. At the "Service" step, enter the name `Billing API`, select a system and a GitLab namespace, and click "Next".
1. At the "Parameters" step, check that only the "HTTP port" field is shown, and click "Create microservice".
1. In the "Run process" dialog, click "Run" and wait for the process to complete.

The template works if the following conditions are met:

- The `billing-api` project with the `cmd/billing-api/main.go` and `.ddp/template.lock.yaml` files is created in GitLab.
- The lock file contains `templateId: go-service` and `templateVersion: 1.0.0`.
- On the microservice card on the Home page, the "Template" and "Version" fields contain `go-service` and `1.0.0`.

Releasing the next template version and upgrading microservices are described in the [template version release example](template-release-update/).
