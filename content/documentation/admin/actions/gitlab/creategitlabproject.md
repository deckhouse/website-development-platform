---
title: CreateGitlabProjecte
weight: 20
---

{{< alert level="info" >}}
Running this action requires the token of the user on whose behalf it will be run.
{{< /alert >}}

CreateGitlabProject — creates a new project in GitLab. The action calls the GitLab API to create a project with the specified parameters, such as name, project path, description, and other settings. A GitLab token, which must be provided in the credentials, is used for authentication.

### Request example

```yaml
name: example
path: example
description: example
default_branch: main
initialize_with_readme: false
namespace_id: '0'
```

### Request specification

| Name                     | Required | Description                                                                           | Default value |
| ------------------------ | -------- | ---------------------------------------------------------------------------------------- | ----------------------- |
| name                     | Yes      | Name of the project to be created in GitLab                                               | -                       |
| path                     | Yes      | URL-friendly project path. Usually matches the name, but may differ                       | -                       |
| default_branch           | Yes      | Name of the branch to be used by default, e.g. "main"                                     | -                       |
| namespace_id             | Yes      | Identifier of the GitLab namespace in which the project will be created                   | -                       |
| initialize_with_readme   | No       | Flag indicating whether the project should be initialized with a README file              | false                   |
| description              | No       | Project description visible to users                                                      | -                       |

### Note

The action performs a POST request to the URL: `/api/v4/projects`, sending the request parameters in JSON format. Upon successful creation, GitLab returns data about the newly created project.
