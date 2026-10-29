---
title: Bitbucket. Tags
description: Display and creation settings for tags in a Bitbucket repository.
weight: 20
---

The widget displays data about tags in a Bitbucket repository.

## Configuration

When adding the widget, specify the following parameters:

| Name        | Required | Description                                                    | Example                                                    |
| ----------- | -------- | -------------------------------------------------------------- | ---------------------------------------------------------- |
| Project key | Yes      | The part of the repository URL immediately after `/projects/` | For `https://<BITBUCKET_HOST>/projects/MYTEAM/repos/backend`, specify `MYTEAM` |
| Repository identifier | Yes | The part of the repository URL immediately after `/repos/`    | For `https://<BITBUCKET_HOST>/projects/MYTEAM/repos/backend`, specify `backend` |

where:

- `<BITBUCKET_HOST>`: The hostname of the Bitbucket server.

## Displayed data

For each repository tag, the widget displays the tag name and commit details,
including the hash, message, author, creation date, and a link to the commit in Bitbucket.

## Additional widget features

In addition to displaying data, the widget supports the actions described below.

### Creating tags

The widget can create tags in Bitbucket directly from Deckhouse Development Portal (DDP).

#### Configuration

When adding the widget, specify the following parameters:

| Name        | Required | Description                                                                  |
| ----------- | -------- | ---------------------------------------------------------------------------- |
| Name        | Yes      | The name of the tag to create                                               |
| Create from | Yes      | The branch or existing tag from which to create the new tag. Select it from the list |
| Description | No       | The description of the tag to create                                        |

## Authorization

Authentication is described in [External services](../../external-services/#bitbucket).
