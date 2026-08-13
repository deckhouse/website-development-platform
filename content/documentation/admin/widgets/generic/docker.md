---
title: Docker
description: Configure the Docker widget to browse container images, tags, and pull commands in a registry.
weight: 70
---

The widget displays available images in a Docker registry. It shows all available tags and the `docker pull` command. Search is supported.

## Configuration

| Name | Required | Description                                                                                                            | Default |
| ---- | -------- | ---------------------------------------------------------------------------------------------------------------------- | ------- |
| URL  | Yes      | Docker Registry URL used to retrieve available image data                                                              | —       |
| Name | No       | Name of the repository from which the widget loads data. Example: `repo`. If omitted, all available images are loaded | —       |


## Authorization

Authorization is configured in [External services](../../external-services/#docker-registry).
