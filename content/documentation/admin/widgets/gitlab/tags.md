---
title: GitLab. Tags
weight: 70
---

The widget displays data about GitLab project tags.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.

## Configuration

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | -             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | -             |

## Additional widget features

### Creating tags

The widget lets you create GitLab tags directly from DDP.

#### Configuration

| Name        | Required | Description                                          | Default value |
| ----------- | -------- | ---------------------------------------------------- | ------------- |
| Name        | Yes      | Name of the tag to create                            | -             |
| Create from | Yes      | Branch or existing tag from which to create the tag  | -             |
| Description | No       | Description of the tag to create                     | -             |
