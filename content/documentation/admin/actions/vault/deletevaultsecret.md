---
title: DeleteVaultSecret
weight: 20
---


{{< alert level="info" >}}
Running this action requires a Vault token with permission to create secrets.
{{< /alert >}}

DeleteVaultSecret — deletes a secret from HashiCorp Vault.

### Request example

```yaml
path: example/data/path
```

### Request specification

| Name                        | Required | Description                                                                                             |
| --------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| path                        | Yes      | Path at which the secret to be deleted is located in Vault                                              |
