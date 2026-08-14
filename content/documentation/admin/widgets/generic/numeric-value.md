---
title: Numeric value
description: Display a numeric value derived from static data or a template.
weight: 170
---

The widget displays a specified numeric value.

## Configuration

| Name          | Required | Description                                                                                                                                                          | Default |
| ------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| Resource      | No       | Resource from which required values are extracted when processing the template                                                                                       | —       |
| Numeric value | No       | Value displayed in the widget. Templating is supported. Without templating: `100`. With templating: `{{ .entity.properties.id }}`                                   | —       |
