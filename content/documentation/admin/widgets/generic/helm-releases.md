---
title: Helm releases
weight: 20
---

The widget displays data about Helm releases in Kubernetes and lets you roll back to previous versions.

The widget displays:

* **Helm release list** — Information about current releases created with Helm in the specified Kubernetes namespace.
* **Release manifests** — Manifests associated with Helm releases in the specified Kubernetes namespace, including YAML files that define resource configuration and state.
* **Values** — Variables used to deploy Helm releases.

## Authorization

Authorization is configured in [External services](../../external-services/#kubernetes).

## Configuration

| Name      | Required | Description                                                                          | Default |
| --------- | -------- | ------------------------------------------------------------------------------------ | ------- |
| Namespace | No       | Namespace from which the widget loads data. Example: `default`                       | -       |
| Release   | No       | Name of the release from which the widget loads data. Example: `my-release`          | -       |
