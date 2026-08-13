---
title: Kubernetes. Resource quotas
description: Resource quota usage visualization for Kubernetes namespaces.
weight: 40
---

The widget displays resource quota data from a Kubernetes cluster.

For each quota, the widget visualizes the resources in use.

## Configuration

| Name      | Required | Description                                                             | Default value |
| --------- | -------- | ----------------------------------------------------------------------- | ------------- |
| URL       | Yes      | Kubernetes API server URL used to retrieve data from Kubernetes         | —             |
| Namespace | Yes      | Namespace from which the widget loads data. Example: `default`          | —             |


## Authorization

Authorization is described in [External services](../../external-services/#kubernetes).
