---
title: Template pack
weight: 30
description: DDP microservice template pack format — the template.yaml manifest, the values.yaml parameter schema, the x-ddp-from annotation, file templating, versions, and publishing with the ddp-template utility.
---

A template pack is a directory with the files of a future microservice repository, a manifest, and a parameter schema.
The portal renders the pack files with the Go template engine using the parameters that the user fills in when creating a microservice.
The pack is published to a container registry with the `ddp-template` utility.

## Pack structure

A pack contains three service files and the repository files:

- `template.yaml`: Manifest with the ID, name, language, and upgrade rules.
- `values.yaml`: Template parameter schema.
- `.templateignore`: Paths that are copied to the repository without templating.
- Other files and directories: Contents of the microservice repository.

Example structure of a Go service pack:

```text
go-service/
  template.yaml
  values.yaml
  .templateignore
  .gitlab-ci.yml
  cmd/{{ .slug }}/main.go
  go.mod
  deploy/helm/Chart.yaml
```

The `template.yaml`, `values.yaml`, and `.templateignore` files are not copied to the microservice repository.
The `.git` directory is not included in the pack.

Do not place a `.ddp/` directory in the pack, because the portal creates the `.ddp/template.lock.yaml` lock file in it when creating and upgrading the repository.

## The template.yaml manifest

The manifest describes the template and its upgrade rules. Example manifest:

```yaml
id: go-service
name: Go service
description: Go HTTP service with CI and Helm chart
language: Go
tags:
  - http
  - helm
minDdpVersion: "1.7.0"
update:
  strategy: selective_additive
  overwrite:
    - .gitlab-ci.yml
    - deploy/**
    - go.mod
  addIfMissing:
    - .dockerignore
    - .editorconfig
```

Manifest fields:

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Template ID: lowercase Latin letters, digits, and hyphens. Matches the repository name in the container registry |
| `name` | Yes | Template name in the gallery |
| `description` | No | Template description in the gallery |
| `language` | Yes | Template language: `Go`, `Vue`, `TypeScript`, `Python`, `Java`, or `Other` |
| `tags` | No | Tags used for search in the gallery |
| `minDdpVersion` | No | Minimum Deckhouse Development Portal (DDP) version in semver format. On an earlier DDP version, the template version gets the "unavailable" status |
| `update` | Yes | Rules for upgrading the repository to a new template version: the `strategy`, `overwrite`, and `addIfMissing` fields |
| `update.strategy` | No | Upgrade strategy: `selective`, `additive`, `selective_additive`, `full`, or `merge`. If not set, `selective` is used, and `update.overwrite` becomes required. The `full` strategy requires no path lists: `update: {strategy: full}` |
| `update.overwrite` | Depends on the strategy | Glob patterns of paths that are overwritten during an upgrade |
| `update.addIfMissing` | Depends on the strategy | Glob patterns of paths that are added during an upgrade if they are missing from the repository |

No other fields are allowed in the manifest.
Strategies and path list requirements are described in ["Upgrade strategies"](updates/#upgrade-strategies).

## The values.yaml parameters

The `values.yaml` file is the template parameter schema in OpenAPI Schema Object format.
The portal uses the schema to build the form on the "Parameters" step of the microservice creation wizard.

Example parameter schema:

```yaml
type: object
required:
  - slug
  - name
  - port
properties:
  slug:
    type: string
    title: Service slug
    description: Identifier used in paths and module name
    x-ddp-from: context.microservice.slug
  name:
    type: string
    title: Display name
    x-ddp-from: context.microservice.name
  port:
    type: integer
    title: HTTP port
    default: 8080
  database:
    type: string
    title: Database
    enum:
      - postgres
      - none
    default: none
  metrics:
    type: boolean
    title: Expose Prometheus metrics
    default: true
```

The portal uses the top-level `properties` to build the form:

- `title`: Field label. If not set, the parameter key is used.
- `description`: Field hint.
- `required`: Whether the field is required in the form.
- `default`: Initial value.
  If no default value is set, the empty value of the type is used: an empty string, `0`, `false`, an empty list, or an empty object.

The form field type depends on the parameter type:

| Parameter type | Form field |
| --- | --- |
| `string` with `enum` | Drop-down list |
| `string` | Text field |
| `boolean` | Toggle |
| `integer` | Numeric field with a step of 1 |
| `number` | Numeric field with a step of 0.1 |
| `object`, `array`, schemas with `oneOf`, `anyOf`, `allOf`, `$ref` | JSON input field |

Before rendering the pack, the portal validates the resulting values against the schema.
If they do not match, creation fails with the `values_invalid: values do not match values.yaml schema: ...` error.

When the pack is rendered, unset parameters get the empty values of their types, so `required` does not guarantee a non-empty value.
To prohibit an empty string, use schema constraints, for example, `minLength: 1`.

The schema is defined as an object. A list of parameters in `values.yaml` is not supported.

### Binding parameters to the launch context

The `x-ddp-from` annotation fills a parameter from the launch context of the microservice creation process.
Such a parameter is not shown in the wizard form, so the user does not re-enter data already provided on the "Service" step.

The annotation value is a path in the launch context that starts with `context.`:

| Path | Value |
| --- | --- |
| `context.microservice.slug` | Microservice ID |
| `context.microservice.name` | Microservice name |
| `context.system.slug` | ID of the system that the microservice belongs to |
| `context.system.name` | System name |
| `context.repository.default_branch` | Default branch |
| `context.repository.description` | Repository description |
| `context.template.version` | Template version |

The full contents of the creation context are described in ["Home page processes"](../home/processes/#creating-a-microservice).

Annotation rules:

- The value from the context replaces the value passed in the process parameters.
- If the path is missing from the context or its value is `null`, creation fails with the `values_invalid: values.yaml property "k": x-ddp-from "context.x": path not found in launch context` error.
- If the annotation is empty, creation fails with the `values_invalid: values.yaml property "k": x-ddp-from is empty` error.
- The annotation is applied only when the repository is created. During an upgrade, the values from the lock file are used.

## File templating

Pack files are rendered with the Go `text/template` engine.
Template parameters are available at the root: `{{ .slug }}`, `{{ .port }}`.

Example pack file:

```go-html-template
package main

// {{ .name }} listens on port {{ .port }}.
const port = {{ .port }}
{{- if .metrics }}

const metricsPath = "/metrics"
{{- end }}
```

Templating rules:

- Referencing a missing parameter makes rendering fail.
- Templates can use the [portal templating functions](../../user/templating/#built-in-functions) and Sprig library functions, except `env`, `expandenv`, and `getHostByName`.
- File and directory names are templated, for example, `cmd/{{ .slug }}/main.go`.
  A path that points outside the repository after rendering makes rendering fail.
- A file that is empty or contains only whitespace after rendering is not written to the repository.
  This is how optional files are defined in a pack.

### Files without templating

The `.templateignore` file lists paths that are copied to the repository as is, without templating.
Use it for files with their own `{{ }}` syntax, for example, Helm templates.

Example `.templateignore`:

```text
# Helm templates use their own {{ .Values }} syntax.
deploy/helm/templates/**
```

Each line is a glob pattern of a path relative to the pack root. The `**` syntax is supported.
Lines that start with `#` are comments.
Path patterns in `.templateignore` can contain `{{ }}` expressions themselves.

## Versions

A pack version is an artifact tag in the container registry in strict semver format without the `v` prefix, for example, `1.2.0` or `2.0.0-rc.1`.

Versioning rules:

- A published version cannot be overwritten. To fix it, publish a new version.
- Increment the major version for breaking changes to the repository structure.
- Prerelease versions, for example, `2.0.0-rc.1`, are not installed by automatic upgrades and are visible in the microservice card only to users with the `read:template-registries` permission.

Before publishing a version with breaking changes, check the following:

- Files renamed or moved in the new version remain at the old paths in microservice repositories, because an upgrade does not delete files.
  Describe the manual cleanup in the version description.
- New required files are added to `update.overwrite` or `update.addIfMissing` if users may already have their own copies of these files.
- `update.overwrite` contains only files owned by the platform, for example, CI configuration and deployment manifests.
  Move files that developers edit to `update.addIfMissing` or leave them out of the lists.

## The ddp-template utility

The `ddp-template` utility validates, builds, publishes, and renders template packs locally.

Utility commands:

| Command | Purpose |
| --- | --- |
| `ddp-template lint <PATH>` | Validates `template.yaml`, `values.yaml`, `.templateignore`, and the upgrade path lists, and performs a trial render of the pack |
| `ddp-template pack <PATH>` | Builds a pack archive without the `.git` directory |
| `ddp-template publish <PATH> --version <VERSION>` | Validates, builds, and publishes the pack to the container registry. Outputs JSON with the `templateId`, `version`, `ociRef`, and `sha256` fields |
| `ddp-template pull <TEMPLATE_ID> --version <VERSION>` | Downloads and unpacks a published pack |
| `ddp-template render <PATH>` | Renders the pack locally, including simulation of an upgrade strategy |

Exit codes of `ddp-template lint`: `0` if there are no errors, `1` if there are errors, `2` if there are warnings.
The `--strict` flag turns warnings into errors, and the `--render-smoke` flag performs only rendering with default values.

### Connecting to the registry

Registry connection parameters are set with flags, environment variables, or a configuration file.
Flags take precedence over environment variables, and environment variables take precedence over a profile from the configuration file.

| Parameter | Flag | Environment variable |
| --- | --- | --- |
| Configuration file | `--config` | `DDP_TEMPLATE_CONFIG` |
| Profile | `--profile` | `DDP_TEMPLATE_PROFILE` |
| Registry host | `--registry` | `DDP_TEMPLATE_REGISTRY` |
| Repository prefix | `--repository-prefix` | `DDP_TEMPLATE_REPOSITORY_PREFIX` |
| Username | `--registry-user` | `DDP_TEMPLATE_REGISTRY_USER` |
| Password or token | `--registry-token` | `DDP_TEMPLATE_REGISTRY_TOKEN` |

If no username is set, the password is used as a Bearer token.

The configuration file contains connection profiles:

```yaml
default: company
registries:
  company:
    registry: registry.example.com
    repositoryPrefix: ddp-templates
    auth:
      username: <REGISTRY_USER>
      password: <REGISTRY_TOKEN>
```

Where:

- `<REGISTRY_USER>`: Registry username.
- `<REGISTRY_TOKEN>`: Registry password or token.

If the `default` parameter is set or there is only one profile, that profile is used when no profile is specified.

### Publishing from CI

Versions are published from CI by a Git tag.
Example of a GitLab CI job that publishes the `go-service` template by the `go-service/vX.Y.Z` tag:

```yaml
publish-go-service:
  rules:
    - if: $CI_COMMIT_TAG =~ /^go-service\/v\d+\.\d+\.\d+$/
  script:
    - VERSION="${CI_COMMIT_TAG#go-service/v}"
    - ddp-template publish ./template-packs/go-service
        --version "$VERSION"
        --registry "$REGISTRY"
        --repository-prefix ddp-templates
```

After publishing, the version appears in the portal on the next template registry sync or after you click "Sync" on the "Sync runs" tab.
If the "Repository IDs" field is filled in for the connection, add the new template ID to it.

### Checking an upgrade locally

The utility renders a new pack version with the values of an existing microservice without accessing GitLab.
This lets you check which files an upgrade will change.

The following commands download a published version and render it with a simulated upgrade strategy:

```shell
ddp-template pull go-service --version 1.1.0 \
  --registry registry.example.com \
  --repository-prefix ddp-templates \
  --output ./packs/go-service-1.1.0
ddp-template render ./packs/go-service-1.1.0 \
  --values ./values.yaml \
  --use-overwrite \
  --output ./render-out
```

Where `./values.yaml` is a file with the parameter values from the `values` section of the microservice lock file.

The `ddp-template render` command does not apply `x-ddp-from` annotations.
Pass the values of such parameters in the `--values` file.
