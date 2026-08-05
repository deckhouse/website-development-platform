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
