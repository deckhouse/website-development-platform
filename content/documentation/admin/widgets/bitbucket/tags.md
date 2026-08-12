---
title: Bitbucket. Tags
weight: 20
---

The widget displays data about tags in a Bitbucket repository.

## Authentication

Authentication is described in [External services](../../external-services/#bitbucket).

## Configuration

| Name        | Required | Description                                                    | Example                                                    |
| ----------- | -------- | -------------------------------------------------------------- | ---------------------------------------------------------- |
| Project key | Yes      | The part of the repository URL immediately after `/projects/`. | For `.../projects/MYTEAM/repos/backend`, specify `MYTEAM`.  |
| Repository  | Yes      | The part of the repository URL immediately after `/repos/`.    | For `.../projects/MYTEAM/repos/backend`, specify `backend`. |

## Displayed data

The widget displays a list of repository tags with the following information:

- **Tag name** — the tag name.
- **Commit** — the commit hash, message, author, creation date, and a link to the commit in Bitbucket.

## Additional widget features

### Creating tags

The widget can create tags in Bitbucket directly from DDP.

#### Configuration

| Name        | Required | Description                                                                  |
| ----------- | -------- | ---------------------------------------------------------------------------- |
| Name        | Yes      | The name of the tag to create.                                               |
| Create from | Yes      | The branch or existing tag from which to create the new tag. Select it from the list. |
| Description | No       | The description of the tag to create.                                        |
