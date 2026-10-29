---
title: Numeric value
description: Display a numeric value derived from static data or a template.
weight: 170
---

The widget displays a specified numeric value.

## Configuration

When adding the widget, specify the following parameters:

| Name          | Required | Description                                                                                                                                                          | Default value |
| ------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| Number        | No       | Value displayed in the widget. Templating is supported. Without templating: `100`. With templating: `{{ .entity.properties.id }}`                                   | —       |
