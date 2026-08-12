---
title: GitHub. Tags
weight: 30
---

The widget displays GitHub repository tags with commit information, including the author, date, and description. It can also create tags.

## Authentication

Authentication is described in [External services](../../external-services/#github).
In the external service settings, set **URL** to `https://api.github.com`.

## Account and action initiator

Requests to GitHub use the token from the credentials of the platform user on whose behalf the action is invoked. If **Select an account for the widget** is enabled in the widget settings, the selected platform user's credentials are used instead of the current user's credentials.

For an annotated tag, where **Description** is provided, the annotation author fields (`tagger`) in the Git tag metadata are populated with the name and email address of the platform user who performed the action, as defined in the DDP profile. If no name is specified, the email address may be used.

For a lightweight tag without a description, no separate Git tag author is set. The tag is created as a reference to a commit.

The API creates the tag using the GitHub account associated with the token. However, the `tagger` data comes from the DDP profile and may not match the GitHub login.

## Configuration

| Name             | Required | Description                                           | Example                                                       |
| ---------------- | -------- | ----------------------------------------------------- | ------------------------------------------------------------- |
| Repository owner | Yes      | Repository owner, either an organization or a user.   | For `https://github.com/example/my-repo`, specify `example`.   |
| Repository       | Yes      | Repository name without the `.git` suffix.            | For `https://github.com/example/my-repo`, specify `my-repo`.   |

## Displayed data

The table displays the tag, description, commit author, commit link, and commit creation date. The **View** action displays the commit description for each tag.

## Additional widget features

### Creating a tag

The widget can create tags in GitHub. Specify the following fields in the **Create tag** dialog:

| Name        | Required | Description                                                                                       | Default value |
| ----------- | -------- | ------------------------------------------------------------------------------------------------- | ------------- |
| Tag name    | Yes      | A unique tag name, such as `v1.0.0` or `release-2024-01`.                                         | —             |
| Create from | Yes      | The branch or existing tag from which to create the new tag.                                      | —             |
| Description | No       | Tag annotation, such as a release description. If specified, the widget creates an annotated tag. | —             |

{{< alert level="info" >}}
Creating tags requires write permissions in the GitHub repository.
{{< /alert >}}
