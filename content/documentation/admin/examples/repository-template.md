---
title: Repository template in DDP
---

This guide explains how to create a repository template for the DDP platform. A repository template is a Git repository that contains templated code and can be used to create new repositories by applying templating variables.

## What is a repository template

A **repository template** is a dedicated Git repository that contains code with variables written in the [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax) format. When you create a new repository from a template, the DDP platform:

- Clones the template repository.
- Applies templating variables to both file contents and file names.
- Creates a new repository with the final (rendered) version of the code.

Repository templates help standardize project structure, speed up the creation of new services, and maintain consistent code organization practices across teams.

## Purpose and benefits

Repository templates in DDP provide the following benefits:

- **Project standardization**: a unified structure, conventions, and baseline configuration for all repositories created from the template.
- **Service creation automation**: fast creation of new repositories with a preconfigured structure.
- **Centralized management**: changes to the template can be automatically propagated to repositories created from it.
- **Fewer errors**: minimized manual steps when creating new projects.
- **Faster onboarding**: new team members get a ready-to-use structure, examples, and documentation in one place.

## How templating works

The DDP platform uses a templating mechanism based on [Go template](https://developer.hashicorp.com/nomad/tutorials/templates/go-template-syntax) to process template repositories.

Key templating capabilities:

- **File content templating**: variables in the `{{ .variableName }}` format are replaced with values during rendering.
- **File and directory name templating**: file and directory names can include template expressions.
- **Conditional logic**: `{{ if }}`, `{{ else }}`, `{{ end }}` constructs can be used to conditionally include content.
- **Templating functions**: support for functions from the [Sprig](https://masterminds.github.io/sprig/) library for working with strings, numbers, dates, and other data types.

## Creating a repository template

To create a repository template, follow these steps:

1. Create a new Git repository in your version control system (for example, GitLab). This repository will serve as the template.

   Recommendations:

   - Use a clear name that reflects the template purpose (for example, `service-template`, `microservice-template`).
   - Place the template in a dedicated group or namespace to simplify management.
   - Configure repository access according to your organization's security policies.

1. Create the directory and file structure that will be used in projects generated from the template. File and directory names may include template expressions. The structure can be adapted to any programming language or project type.

   Example structure for a Go project:

   ```sh
   ├── src/
   │   └── {{ .moduleName }}/
   │       ├── main.go
   │       └── config.go
   ├── tests/
   │   └── {{ .moduleName }}_test.go
   ├── docs/
   │   └── README.md
   ├── .gitignore
   ├── values.yaml
   └── .templateignore
   ```

   Example structure for a Python project:

   ```sh
   ├── {{ .packageName }}/
   │   ├── __init__.py
   │   └── main.py
   ├── tests/
   │   └── test_{{ .packageName }}.py
   ├── requirements.txt
   ├── README.md
   ├── .gitignore
   ├── values.yaml
   └── .templateignore
   ```

   Example structure for a web application:

   ```sh
   ├── {{ .appName }}/
   │   ├── components/
   │   ├── pages/
   │   └── utils/
   ├── public/
   ├── package.json
   ├── README.md
   ├── .gitignore
   ├── values.yaml
   └── .templateignore
   ```

1. Add template variables to your files. Use Go template syntax to parameterize file contents. Templating is applied to files of any type and in any programming language.

   Example Go file `src/{{ .moduleName }}/main.go`:

   ```go
   package {{ .moduleName }}

   import (
       "fmt"
       "log"
   )

   func main() {
       log.Printf("Starting service: %s", "{{ .serviceName }}")
       log.Printf("Version: %s", "{{ .version }}")

       {{- if .enableMetrics }}
       log.Println("Metrics enabled")
       {{- end }}

       fmt.Println("Service is running")
   }
   ```

   Example Python file `{{ .packageName }}/main.py`:

   ```python
   """
   {{ .serviceName }}
   {{ .description }}
   """

   import logging

   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)

   def main():
       logger.info(f"Starting service: {{ .serviceName }}")
       logger.info(f"Version: {{ .version }}")
       
       {{- if .enableMetrics }}
       logger.info("Metrics enabled")
       {{- end }}
       
       print("Service is running")

   if __name__ == "__main__":
       main()
   ```

   Example configuration file `config.yaml`:

   ```yaml
   service:
     name: "{{ .serviceName }}"
     version: "{{ .version }}"
     {{- if .enableMetrics }}
     metrics:
       enabled: true
     {{- end }}
   ```

   Example `README.md` file:

   ```markdown
   # {{ .serviceName }}

   {{ .description }}

   ## Configuration

   - Module name: `{{ .moduleName }}`
   - Version: `{{ .version }}`
   {{- if .enableMetrics }}
   - Metrics: enabled
   {{- end }}
   ```

1. Create the `values.yaml` file in the repository root to define default variable values. The file is optional, but recommended because it provides sensible defaults and documents the available variables. Variables defined in `values.yaml` can be referenced from files of any type and in any programming language.

   > If `values.yaml` is missing, all required variables must be provided when running the action that creates a repository from a template. If a variable is not provided either in `values.yaml` or in the action parameters, rendering will fail with an error if the template references that variable.

   Example `values.yaml`:

   ```yaml
   # Module name (used in code).
   moduleName: example-module

   # Service name (used in documentation).
   serviceName: Example Service

   # Service description.
   description: This is an example service created from template

   # Service version.
   version: 1.0.0

   # Enable metrics.
   enableMetrics: false
   ```

1. If the template contains files or directories that include template expressions but must not be processed by the templating engine (for example, Helm charts that use Go templates themselves), create a `.templateignore` file in the repository root.

   Example `.templateignore`:

   ```sh
   # Exclude Helm charts from templating.
   helm/**
   charts/**

   # Exclude documentation that may contain template examples.
   docs/examples/**
   ```

1. Before using the template in a production environment, it is recommended to test it locally with the `ddp-render-dir` utility. This tool helps validate that the template renders correctly with different variable values.

   Example:

   ```bash
   ddp-render-dir \
     --source-dir ./template-repo \
     --target-dir ./rendered-output \
     --values ./test-values.yaml
   ```

1. After preparing the template, commit all changes and push them to the repository:

   ```bash
   git add .
   git commit -m "Initial template structure"
   git push origin main
   ```

   Recommendations for versioning:

   - Use [Semantic Versioning](https://semver.org/) for template tags (for example, `v1.0.0`, `v1.1.0`).
   - Create tags for stable template releases.
   - Versioning helps track which template version was used to create a repository.

## Examples of using variables

Templating works with files of any type and programming language. Below are examples of using variables in different contexts.

### Basic variables

Use simple variables to substitute values in code:

- Example for Go:

  ```go
  // In the template
  package {{ .packageName }}

  const ServiceName = "{{ .serviceName }}"
  ```

  With `packageName: "api"` and `serviceName: "user-service"`, the rendered result will be:

  ```go
  package api

  const ServiceName = "user-service"
  ```

- Python example:

  ```python
  # In the template
  PACKAGE_NAME = "{{ .packageName }}"
  SERVICE_NAME = "{{ .serviceName }}"
  ```

  With `packageName: "api"` and `serviceName: "user-service"`, the rendered output will be:

  ```python
  PACKAGE_NAME = "api"
  SERVICE_NAME = "user-service"
  ```

- Example for configuration file:

  ```yaml
  # In the template
  app:
    name: "{{ .appName }}"
    version: "{{ .version }}"
  ```

  With `appName: "user-service"` and `version: "1.0.0"`, the rendered output will be:

  ```yaml
  app:
    name: "my-app"
    version: "1.0.0"
  ```

### Conditional logic

Using conditions to include or exclude code depending on variable values:

```yaml
# values.yaml
enableDatabase: true
enableCache: false
```

- Example for Go:

  ```go
  // In the template
  func init() {
      {{- if .enableDatabase }}
      initDatabase()
      {{- end }}
      
      {{- if .enableCache }}
      initCache()
      {{- end }}
  }
  ```

  Result with the specified values:

  ```go
  func init() {
      initDatabase()
  }
  ```

- Example for configuration files:

  ```yaml
  # In the template
  database:
    {{- if .enableDatabase }}
    enabled: true
    host: "{{ .dbHost }}"
    {{- end }}

  cache:
    {{- if .enableCache }}
    enabled: true
    {{- end }}
  ```

- Example for documentation:

  ```markdown
  # {{ .serviceName }}

  {{ .description }}

  {{- if .enableDatabase }}
  ## Database Configuration

  Database is enabled for this service.
  {{- end }}

  {{- if .enableCache }}
  ## Cache Configuration

  Cache is enabled for this service.
  {{- end }}
  ```

### Templating functions

Using functions from the [Sprig](https://masterminds.github.io/sprig/) library to transform values.

- Example of using functions to transform strings:

  ```go
  // In the template (Go)
  const ModuleName = "{{ .moduleName | upper }}"
  const PackagePath = "{{ .orgName }}/{{ .moduleName | lower }}"
  ```

  With `moduleName: "UserService"` and `orgName: "example"`, the result will be::

  ```go
  const ModuleName = "USERSERVICE"
  const PackagePath = "example/userservice"
  ```

- Example for other languages:

  ```python
  # In the template (Python)
  MODULE_NAME = "{{ .moduleName | upper }}"
  PACKAGE_PATH = "{{ .orgName }}/{{ .moduleName | lower }}"
  ```

  ```yaml
  # In the template (YAML)
  module:
    name: "{{ .moduleName | lower }}"
    display_name: "{{ .moduleName | title }}"
  ```

- Example of using functions to work with numbers and dates:

  ```yaml
  # In the template
  build:
    timestamp: "{{ now | date "2006-01-02T15:04:05Z07:00" }}"
    version: "{{ .majorVersion }}.{{ .minorVersion }}.{{ .patchVersion }}"
  ```

### Path templating

Using variables in file and directory names:

```sh
# Template structure
src/
└── {{ .moduleName }}/
    └── {{ .moduleName }}.go
```

With `moduleName: "auth"`, the result will be:

```sh
src/
└── auth/
    └── auth.go
```

## Using a template in DDP

After you create a repository template, you can use it in the DDP platform to create new repositories. Configuring template usage includes:

1. Registering the template in the DDP catalog: create an entity in the **Templates** resource and specify the template repository URL.
1. Configuring the **Create repository from template** action: set up the action to use your template.
1. Configuring the service creation process: combine actions into a process to automate service creation from the template.

## Recommendations for creating templates

When creating repository templates, follow these best practices:

- **Document variables**: include a `README.md` file in the template that lists all available variables, their purpose, format, and example values.
- **Provide default values**: use `values.yaml` to define sensible defaults and reduce the number of required parameters when creating a repository.
- **Keep it minimal**: include only what is truly needed to start a project; avoid unnecessary files, examples, and scaffolding that won’t be used.
- **Test regularly**: validate the template with different variable sets before publishing a new version.
- **Version properly**: use Git tags for template versions and maintain a `CHANGELOG.md` to record changes and any potential incompatibilities.
- **Maintain backward compatibility**: when updating a template, try to preserve compatibility with existing variables.
