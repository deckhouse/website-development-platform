---
title: GitLab. Releases
description: Release details and release creation settings for GitLab projects.
weight: 60
---

The widget displays GitLab project releases, highlights the latest release,
and shows related information: the tag, commit link, author, publication date,
and a description with Markdown support.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.

## Configuration

| Name       | Required | Description                                                        | Default value |
| ---------- | -------- | ------------------------------------------------------------------ | ------------- |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Additional widget features

### Creating a release

The widget lets you create a release in GitLab directly from Deckhouse Development Platform (DDP):

| Name         | Required | Description                                                                 | Default value |
| ------------ | -------- | --------------------------------------------------------------------------- | ------------- |
| Release name | Yes      | Release name displayed in the list                                          | —             |
| Tag          | Yes      | Existing tag on which to base the release, selected from the project's tags | —             |
| Description  | No       | Release description in Markdown format                                      | —             |

The created release appears in the list automatically, and the latest release is highlighted.
