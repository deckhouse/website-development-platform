---
title: DeleteVaultSecret
weight: 20
---

{{< alert level="info" >}}
Running this action requires a Vault token with permission to read and delete the secret at the specified path.
For a KV v2 path (contains `/data/`), the token also needs permission to delete the corresponding `/metadata/` path.
{{< /alert >}}

DeleteVaultSecret — deletes a secret from HashiCorp Vault.

## Request example

Example of the action request body:

```yaml
path: example/data/path
```

## Request specification

The action request body supports the following parameters:

| Name                        | Required | Description                                                                                             |
| --------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| `path`                      | Yes      | Path at which the secret to be deleted is located in Vault                                              |
