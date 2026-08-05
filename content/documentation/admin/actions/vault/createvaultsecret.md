---
title: CreateVaultSecret
weight: 10
---


{{< alert level="info" >}}
Running this action requires a Vault token with the permissions needed to create secrets.
{{< /alert >}}

CreateVaultSecret — creates a secret with one or more values in HashiCorp Vault.

### Request example

```yaml
path: example/data/path
secrets:
  - key: key1
    value: value1
  - key: key2
    value: value2
```

### Request specification

| Name                        | Required | Description                                                          | Possible values                       | Default value            |
| --------------------------- | -------- | ------------------------------------------------------------------- | ------------------------------------- | ----------------------- |
| path                        | Yes      | Path at which the secret will be stored in Vault                      |                                       | -                       |
| allow_update                | No       | Determines the action's behavior when creating or updating a secret   | true, false, merge, merge_or_create   | false                   |
| secrets                     | Yes      | Set of secrets to create in Vault                                     |                                       | -                       |
| secrets.key                 | Yes      | Name (identifier) of the secret                                        |                                       | -                       |
| secrets.value               | Yes      | Value of the secret                                                    |                                       | -                       |

### Note

`allow_update` determines the action's behavior when creating or updating a secret:

- `false` (default) — the action fails with an error if the secret already exists;
- `true` — a new version of the secret is created with the values passed to the action;
- `merge` — only the secret keys specified in the action are updated or created. Existing keys not mentioned in the action are preserved unchanged. If the secret does not yet exist at the path, the action fails with an error;
- `merge_or_create` — behaves like `merge` if the secret already exists; if the secret does not exist at the path, it is created (a full write of the values from the action).
