---
title: Percentage value
weight: 180
---

The widget displays a specified percentage value.

## Configuration

| Name             | Required | Description                                                                                                                                                          | Default |
| ---------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| Resource         | No       | Resource from which required values are extracted when processing the template                                                                                       | -       |
| Percentage value | No       | Value displayed in the widget. Templating is supported. Without templating: `100`. With templating: `{{ .entity.properties.id }}`                                   | -       |
