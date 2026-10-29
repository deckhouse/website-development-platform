---
title: CreateGitlabProject
weight: 20
---

{{< alert level="info" >}}
This action requires a token for the user on whose behalf it will run.
{{< /alert >}}

CreateGitlabProject — creates a new project in GitLab through the GitLab API, using the specified name, project path, description, and other settings.
Authentication uses a GitLab token provided in the credentials.

## Request example

Example of the action request body:

```yaml
name: example
path: example
description: example
default_branch: main
initialize_with_readme: false
namespace_id: '0'
```

## Request specification

The action request body supports the following parameters:

| Name                     | Required | Description                                                                           | Default value |
| ------------------------ | -------- | ---------------------------------------------------------------------------------------- | ----------------------- |
| `name`                   | Yes      | Name of the project to be created in GitLab                                               | —                       |
| `path`                   | Yes      | URL-friendly project path. Usually matches the name, but may differ                       | —                       |
| `default_branch`         | Yes      | Name of the branch to be used by default, for example, `main`                                     | —                       |
| `namespace_id`           | Yes      | Identifier of the GitLab namespace in which the project will be created                   | —                       |
| `initialize_with_readme` | No       | Flag indicating whether the project should be initialized with a README file              | `false`                 |
| `description`            | No       | Project description visible to users                                                      | —                       |

## GitLab API request

The action performs a POST request to the URL: `/api/v4/projects`, sending the request parameters in JSON format.
Upon successful creation, GitLab returns data about the newly created project.
