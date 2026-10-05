---
title: CreateKubernetesResource
weight: 10
---

{{< alert level="info" >}}
This action requires a Kubernetes service account token.
{{< /alert >}}

CreateKubernetesResource — creates one or more resources in a Kubernetes cluster or updates existing resources.

### Request example

```yaml
manifests:
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: example1
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: example2
```

### Request specification

| Name                         | Required | Description                                                                                            |
| ---------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| manifests                    | Yes      | Kubernetes manifests to apply                                                                          |

### Response

The action returns an object with one key per applied manifest. The value of each key is the created or updated object as returned by the Kubernetes API.

The key has the form `<KIND>_<NAMESPACE>_<NAME>` in lowercase, with hyphens (`-`) replaced by underscores (`_`):

- `<KIND>`: Kind of the object, for example `deployment`.
- `<NAMESPACE>`: Namespace of the object. If a namespaced object has no `metadata.namespace`, the action creates it in the `default` namespace. For cluster-wide objects, such as Namespace, this part is empty.
- `<NAME>`: Name of the object.

For the request example above, the response contains the `namespace__example1` and `namespace__example2` keys.
For a Deployment `my-app` without `metadata.namespace`, the key is `deployment_default_my_app`, and the template `{{ .response.deployment_default_my_app.metadata.uid }}` returns the object UID.
