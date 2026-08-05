---
title: DeleteKubernetesResource
weight: 30
---

{{< alert level="info" >}}
Running this action requires a Kubernetes service account token.
{{< /alert >}}

DeleteKubernetesResource — deletes an existing resource in a Kubernetes cluster.

### Request example

```yaml
group: apps
version: v1
resource_type: deployments
resource_name: nginx-deployment
namespace: example
```

### Request specification

| Name                        | Required          | Description                                                                    | Possible values                                                                                                                                                                                                                                                                                                                                                                                                                          |
| --------------------------- | ----------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| group                       | Yes                | API group of the resource. Specifies which API group the object being deleted belongs to | [Determining the required Group and Version](getkubernetesresource/#determining-the-required-group-and-version)                                                                                                                                                                                                                                                                                                                          |
| version                     | Yes                | API version of the resource                                                     | [Determining the required Group and Version](getkubernetesresource/#determining-the-required-group-and-version)                                                                                                                                                                                                                                                                                                                          |
| resource_type               | Yes                | Type of the resource to delete                                                  | pods, services, deployments, statefulsets, daemonsets, replicasets, jobs, cronjobs, nodes, namespaces, configmaps, secrets, persistentvolumes, persistentvolumeclaims, limitranges, resourcequotas, horizontalpodautoscalers, ingresses, networkpolicies, serviceaccounts, roles, clusterroles, rolebindings, clusterrolebindings, podsecuritypolicies, storageclasses, volumeattachments, events, endpoints, customresourcedefinitions |
| resource_name               | Yes                | Name of the specific resource to delete                                         | -                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| namespace                   | Yes                | Namespace in which the resource is located                                      | -                                                                                                                                                                                                                                                                                                                                                                                                                                         |
