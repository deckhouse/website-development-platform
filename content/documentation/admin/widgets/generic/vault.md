---
title: Vault Secrets
description: Browse KV v2 secret metadata and key structures without exposing secret values.
weight: 60
---

The widget lets you browse secrets in HashiCorp Vault or Deckhouse Stronghold. KV v2 secrets are supported.

{{< alert level="info" >}}
The widget does not send secret values to users.
Only secret metadata, such as version and creation time, and the key structure without values are sent to the client.
{{< /alert >}}

For each secret, you can:

* Browse the hierarchical structure of secrets and directories.
* View secret metadata: version, creation time, deletion time, and destruction status.
* View secret keys in a key-value table. Placeholders are displayed instead of values.
* Navigate nested secrets and directories.

## Configuration

When adding the widget, specify the following parameters:

| Name      | Required | Description                                                                                                                       | Default value |
| --------- | -------- | --------------------------------------------------------------------------------------------------------------------------------- | ------- |
| Path      | Yes      | Path to a secret or directory in Vault. The path must explicitly include `/data/`. Examples: `services/data/`, `services/data/example` | —    |
| UI prefix | No       | UI URL prefix. Use `vault` for HashiCorp Vault or `stronghold` for Deckhouse Stronghold                                           | —       |

## Displayed data

The widget displays the following information.

### Secret structure

Directories are displayed with a trailing slash, for example, `nested/`, and are always identified as directories even if they contain keys.
Secrets are displayed without a trailing slash and contain keys.

### Secret metadata

The following metadata is displayed when available:

* **Version**: Secret version in KV v2.
* **Creation time**: Date and time when the secret was created.
* **Deletion time**: Date and time when the secret version was deleted.
* **Destruction status**: Indicates that the secret was destroyed.

### Secret keys

Secret keys are displayed in a key-value table.
The **Key** column shows the full path to the key in the secret structure, for example, `database.host`.
The **Value** column is always masked as `********` and cannot be revealed.

## Path behavior

Paths to KV v2 secrets must explicitly include `/data/`. Valid examples:

* `services/data/`: Browse all secrets in the `services` directory.
* `services/data/example`: View the `example` secret.
* `services/data/nested/secret`: View a nested secret.

## Authorization

Authorization is configured in [External services](../../external-services/#vault).
