---
title: Kubernetes. Pods
description: Pod status, specification, and log inspection in Kubernetes.
weight: 30
---

The widget displays information about pods in a Kubernetes cluster.

The following information is available for each pod:

* Pod specification as YAML configuration.
* Container logs.
* Pod state information, including status and restart count.

## Authorization

Authorization is described in [External services](../../external-services/#kubernetes).

## Configuration

| Name           | Required | Description                                                                                                        | Default value |
| -------------- | -------- | ------------------------------------------------------------------------------------------------------------------ | ------------- |
| URL            | Yes      | Kubernetes API server URL used to retrieve data from Kubernetes                                                    | —             |
| Namespace      | No       | Namespace from which the widget loads data. Example: `default`                                                     | —             |
| Label selector | No       | Comma-separated selectors used to filter pods. Example: `app.kubernetes.io/name=example`                           | —             |
