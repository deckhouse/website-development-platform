---
title: Kubernetes. Deployments
description: Deployment inspection, scaling, and resource management in Kubernetes.
weight: 10
---

The Kubernetes deployments widget displays key information about all Deployments in a Kubernetes cluster.
You can filter Deployments by namespace, label selector, or both.

The following actions are available for each Deployment:

- View the Deployment specification and status.
- Scale the number of Deployment replicas. After selecting the required number of replicas, apply the change by clicking the floppy-disk **Save** button.
- View information about pods managed by the Deployment and their containers, including logs for each container.
- View and edit container resources. The widget displays all configured container resources, including CPU, memory, `ephemeral-storage`, and other resource types. You can edit only CPU and memory in the `requests` and `limits` sections. Changes are applied at the Deployment level and propagated to all pods managed by that Deployment. Clearing a CPU or memory value removes the corresponding resource from the container configuration. Other resources, such as `ephemeral-storage`, are displayed but cannot be edited in the widget.

## Configuration

| Name           | Required | Description                                                                                                                                                      | Default value |
| -------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |
| Kubernetes API | Yes      | Kubernetes API server URL used to retrieve data from Kubernetes                                                                                                  | —             |
| Namespace      | No       | Kubernetes namespace from which Deployments are loaded. If no namespace is specified, the widget attempts to load all Deployments in the cluster. Example: `default` | —             |
| Label selector | No       | Comma-separated selectors used to filter Deployments. Example: `app.kubernetes.io/name=example`                                                                  | —             |


## Authorization

Authorization is described in [External services](../../external-services/#kubernetes).
