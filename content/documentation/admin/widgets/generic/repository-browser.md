---
title: Repository browser
description: Browse files and directories from GitLab, Bitbucket, or GitHub repositories.
weight: 190
---

The widget displays the structure and contents of files in GitLab, Bitbucket, and GitHub repositories.

## Configuration

| Name         | Required | Description                                                    | Default |
| ------------ | -------- | -------------------------------------------------------------- | ------- |
| Provider     | Yes      | Service hosting the repository: GitLab, GitHub, or Bitbucket   | —       |
| Branch / Tag | No       | Branch name, tag, or commit SHA                                | `main`  |
| Path         | No       | Directory path in the repository. Leave empty for the root     | —       |
| Recursive    | No       | Retrieve files recursively from subdirectories                 | `false` |

## GitLab configuration

| Name       | Required | Description                              | Default |
| ---------- | -------- | ---------------------------------------- | ------- |
| Project ID | Yes      | GitLab project ID, for example, `12345`  | —       |

## Bitbucket configuration

| Name          | Required | Description                                              | Example | Default |
| ------------- | -------- | -------------------------------------------------------- | ------- | ------- |
| Project key   | Yes      | Bitbucket project key, for example, `MYPROJ`             | `MYPROJ` | —       |
| Repository ID | Yes      | Bitbucket repository ID, for example, `my-repo`          | `my-repo` | —       |

## GitHub configuration

| Name             | Required | Description                                                                                                          | Default |
| ---------------- | -------- | -------------------------------------------------------------------------------------------------------------------- | ------- |
| Repository owner | Yes      | Repository owner: an organization or user. For `https://github.com/example/my-repo`, specify `example`              | —       |
| Repository       | Yes      | Repository name without `.git`. For `https://github.com/example/my-repo`, specify `my-repo`                         | —       |


## Authorization

Configure authorization separately for each provider:

* [GitLab](../../external-services/#gitlab).
* [Bitbucket](../../external-services/#bitbucket).
* [GitHub](../../external-services/#github).
