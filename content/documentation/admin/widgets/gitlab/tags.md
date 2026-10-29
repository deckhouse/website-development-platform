---
title: GitLab. Tags
description: Configuration and creation settings for tags in GitLab projects.
weight: 70
---

The widget displays data about GitLab project tags.

## Configuration

When adding the widget, specify the following parameters:

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                   | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Additional widget features

In addition to displaying data, the widget supports the actions described below.

### Creating tags

The widget lets you create GitLab tags directly from Deckhouse Development Portal (DDP).

#### Configuration

When adding the widget, specify the following parameters:

| Name        | Required | Description                                          | Default value |
| ----------- | -------- | ---------------------------------------------------- | ------------- |
| Name        | Yes      | Name of the tag to create                            | —             |
| Create from | Yes      | Branch or existing tag from which to create the tag  | —             |
| Description | No       | Description of the tag to create                     | —             |

## Authorization

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.
