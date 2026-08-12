---
title: GitLab. Pipeline editor
description: Editing GitLab CI/CD configuration and creating merge requests from the widget.
weight: 30
---

The widget lets you edit the GitLab CI/CD pipeline configuration in `.gitlab-ci.yml`
and create merge requests with the changes.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.

## Configuration

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Displayed data

The widget displays a Monaco code editor for editing `.gitlab-ci.yml`
and a diff view of the original and edited configuration.

## Additional widget features

### Creating a merge request

The widget lets you create a merge request containing pipeline configuration changes.

#### Merge request parameters

| Name            | Required | Description                                           | Default value |
| --------------- | -------- | ----------------------------------------------------- | ------------- |
| MR title        | Yes      | Short title describing the purpose of the merge request | —           |
| MR description  | No       | Detailed description of the merge request and changes | —             |
| New branch name | Yes      | Name of the new branch that will contain the changes  | —             |
| Target branch   | Yes      | Branch to which the merge request will be submitted   | `main`        |
| Commit message  | Yes      | Description of changes to the pipeline configuration  | —             |

### Limitations

- The widget supports only the `.gitlab-ci.yml` file in the project root.
- Creating a merge request requires write access to the repository.
- The maximum configuration file size is limited by the GitLab API.
