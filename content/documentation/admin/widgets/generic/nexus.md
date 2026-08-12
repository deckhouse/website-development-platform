---
title: Nexus
weight: 90
---

The widget displays a list of artifacts in a Nexus repository.

## Authorization

Authorization is configured in [External services](../../external-services/#nexus).

## Configuration

| Name       | Required | Description                                                                           | Default |
| ---------- | -------- | ------------------------------------------------------------------------------------- | ------- |
| URL        | Yes      | Nexus API URL used to retrieve data                                                   | -       |
| Repository | Yes      | Name of the repository whose data is displayed in the widget. Example: `my-repo`     | -       |
| Name       | No       | Name of the artifact whose data is displayed in the widget                            | -       |
