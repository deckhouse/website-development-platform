---
title: CreateVaultKubernetesAuthRole
weight: 30
---

{{< alert level="info" >}}
Running this action requires a Vault token with permissions to create/update Kubernetes auth backend roles.
{{< /alert >}}

CreateVaultKubernetesAuthRole — creates or updates a Kubernetes authentication role in HashiCorp Vault.

### Request example

```yaml
mountPath: kubernetes
role: example
bound_service_account_names:
  - default
bound_service_account_namespaces:
  - default
optional:
  token_ttl: 1h
  token_max_ttl: 12h
  audience: vault
  token_policies:
    - default
```

### Request specification

| Name                                  | Required | Description                                                                          |
| -------------------------------------- | -------- | ------------------------------------------------------------------------------------- |
| mountPath                             | Yes      | Mount path of the Kubernetes auth backend in Vault (e.g., kubernetes)                   |
| role                                  | Yes      | Name of the role to create in Vault                                                     |
| bound_service_account_names           | Yes      | List of service account names allowed to access via this role                           |
| bound_service_account_namespaces      | Yes      | List of namespaces allowed to access via this role                                      |
| optional                              | No       | Additional role parameters (listed in the table below)                                  |

Supported values in optional:

| Field                     | Type            | Description                                                              |
| ------------------------- | --------------- | ----------------------------------------------------------------------- |
| token_ttl                 | string          | Time-to-live (TTL) of the token issued at login                          |
| token_max_ttl             | string          | Maximum TTL of the token                                                  |
| token_policies            | []string        | Additional policies assigned at login                                    |
| audience                  | string          | Value of the JWT audience (aud) that Vault expects from the token        |
| token_period              | string          | Token renewal period                                                     |
| token_explicit_max_ttl    | string          | Explicit upper bound on the token's TTL                                  |
| token_num_uses            | int             | Limit on the number of times the token can be used                       |
| token_type                | string          | Type of token issued (e.g., service, batch)                              |
| alias_name_source         | string          | Source of the alias name for identity                                    |
| token_no_default_policy   | bool            | Exclude the default policy from the token                                |
| token_bound_cidrs         | []string        | Restriction on the CIDR ranges from which the issued token can be used   |

The full list of supported parameters is provided in the official [HashiCorp Vault documentation](https://developer.hashicorp.com/vault/docs/auth/kubernetes#parameters).
