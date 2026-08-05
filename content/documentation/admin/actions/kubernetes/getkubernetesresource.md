---
title: GetKubernetesResource
weight: 20
---


{{< alert level="info" >}}
This action requires a Kubernetes service account token.
{{< /alert >}}

GetKubernetesResource — retrieves a resource from a Kubernetes cluster.

### Request example

```yaml
group: managed-services.deckhouse.io
version: v1alpha1
resource_type: postgres
resource_name: example-postgres
namespace: default
```

### Request specification

| Name                        | Required | Description                                                                        | Possible values                                                                  |
| --------------------------- | -------- | ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| group                       | Yes      | API group of the resource. Specifies the API group to which the requested object belongs | [Determining the required Group and Version](#determining-the-required-group-and-version) |
| version                     | Yes      | API version of the resource                                                           | [Determining the required Group and Version](#determining-the-required-group-and-version) |
| resource_type               | Yes      | Type of resource to retrieve                                                           | -                                                                                 |
| resource_name               | Yes      | Name of the resource to retrieve                                                       | -                                                                                 |
| namespace                   | Yes      | Namespace containing the resource                                                     | -                                                                                 |

### Response

On success, the action returns the resource object in the `resource` field. If the resource is not found, the action fails with an error.

| Name         | Description                                      |
| ------------ | --------------------------------------------------- |
| `resource`   | Resource object in Kubernetes format              |

### Determining the required Group and Version

Each resource type has its own API group (Group) and version (Version).
A complete list of API resources, groups, and versions is available [in the Kubernetes documentation](https://kubernetes.io/docs/reference/kubernetes-api/).

If you do not know which API groups and versions are required, you can look up the current values.
There are several ways to determine them:

#### Using the `d8 k` utility

The `d8 k explain` command shows the `apiVersion` for a resource.

Example:

```bash
d8 k explain deployment
```

Example output:

```yaml
GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.
    
FIELDS:
...
```

#### Using the documentation

1. Find the resource you need (for example, Deployment).
1. Find the API group and version in the header. For example, for Deployment:

   ```yaml
   apiVersion: apps/v1
   ```

   Here:
    * `API Group` is `apps`;
    * `Version` is `v1`.
