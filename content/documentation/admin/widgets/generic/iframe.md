---
title: Iframe
description: Configure an Iframe widget that displays content from an external URL.
weight: 120
---

{{< alert level="warning" >}}
The Iframe widget works only when `allowIframe: true` is enabled in the security headers configuration (`security.headers.csp.allowIframe`).
This option is disabled by default, so the widget does not display content until the configuration is changed.
{{< /alert >}}

The widget displays data from an external source.

## Configuration

When adding the widget, specify the following parameters:

| Name | Required | Description                                         | Default value |
| ---- | -------- | --------------------------------------------------- | ------- |
| URL  | Yes      | External source URL used to display data in the widget | —    |
