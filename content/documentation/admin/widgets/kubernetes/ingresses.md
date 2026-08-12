---
title: Kubernetes. Ingresses
description: Ingress specification, rule, and TLS inspection in Kubernetes.
weight: 20
---

The widget displays information about Ingress resources in a Kubernetes cluster.

The following information is available for each Ingress:

* Ingress specification as YAML configuration.
* Ingress rules.
* TLS settings.

## Authorization

Authorization is described in [External services](../../external-services/#kubernetes).

## Configuration

| Name           | Required | Description                                                                                                                                              | Default value |
| -------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |
| URL            | Yes      | Kubernetes API server URL used to retrieve data from Kubernetes                                                                                          | —             |
| Namespace      | No       | Kubernetes namespace from which Ingress resources are loaded. If no namespace is specified, the widget attempts to load all Ingress resources in the cluster. Example: `default` | —             |
| Label selector | No       | Comma-separated selectors used to filter Ingress resources. Example: `app.kubernetes.io/name=example`                                                    | —             |
