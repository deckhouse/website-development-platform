---
title: DeleteKubernetesResource
weight: 30
---

{{< alert level="info" >}}
This action requires a Kubernetes service account token.
{{< /alert >}}

DeleteKubernetesResource — deletes an existing resource in a Kubernetes cluster.

### Request example

```yaml
group: apps
version: v1
resource_type: deployments
resource_name: nginx-deployment
namespace: example
ignore_not_found: true
```

### Request specification

| Name                        | Required          | Description                                                                    | Possible values                                                                                                                                                                                                                                                                                                                                                                                                                          |
| --------------------------- | ----------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| group                       | Yes                | API group of the resource. Specifies the API group to which the object being deleted belongs | [Determining the required Group and Version](getkubernetesresource/#determining-the-required-group-and-version)                                                                                                                                                                                                                                                                                                                          |
| version                     | Yes                | API version of the resource                                                     | [Determining the required Group and Version](getkubernetesresource/#determining-the-required-group-and-version)                                                                                                                                                                                                                                                                                                                          |
| resource_type               | Yes                | Type of resource to delete                                                       | pods, services, deployments, statefulsets, daemonsets, replicasets, jobs, cronjobs, nodes, namespaces, configmaps, secrets, persistentvolumes, persistentvolumeclaims, limitranges, resourcequotas, horizontalpodautoscalers, ingresses, networkpolicies, serviceaccounts, roles, clusterroles, rolebindings, clusterrolebindings, podsecuritypolicies, storageclasses, volumeattachments, events, endpoints, customresourcedefinitions |
| resource_name               | Yes                | Name of the resource to delete                                                   | -                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| namespace                   | Yes                | Namespace containing the resource                                               | -                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ignore_not_found            | No                | Treat the deletion as successful when the object is already absent            | `true`, `false`                                                                                                                                                                                                                                                                                                                                                                                                                              |

### Note

By default, deleting a missing object fails. Setting `ignore_not_found: true` makes the action idempotent: an object that is already gone is treated as being in the desired state. This is useful in deletion processes that may be retried. The parameter does not suppress any other Kubernetes API error.
