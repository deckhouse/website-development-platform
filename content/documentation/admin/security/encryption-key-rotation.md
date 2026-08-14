---
title: Encryption key rotation
description: Safely re-encrypt stored credentials and replace the DDP Backend encryption key.
weight: 35
---

Deckhouse Development Platform (DDP) lets you securely replace the encryption key (`security.secretKey`) without losing credentials or other encrypted values. The operation runs from the web interface and re-encrypts all data stored with the current key.

## Reasons to rotate the key

Rotate the encryption key when you need to replace `security.secretKey` in the DDP Backend configuration, for example, to comply with a security policy or after the key has been compromised.

{{< alert level="warning" >}}
Do not change `security.secretKey` in the configuration or restart the backend before re-encryption finishes in the web interface. Otherwise, the stored values will become unavailable.
{{< /alert >}}

## Access permissions

Key rotation requires the global `rotate:encryption-key` permission.

## Data that is re-encrypted

The operation affects the following data categories:

- "User credentials" — values in PostgreSQL for credential types that use the "Database" storage.
- "AI provider credentials" — personal credentials for AI integrations.
- "Temporary action responses" — encrypted temporary responses in action execution records.
- "Vault secrets" — credential values stored in HashiCorp Vault or Deckhouse Stronghold.
- "Vault configuration" — the AppRole Role ID and AppRole Secret ID in the Vault integration settings.

## Procedure

1. Go to "Administration" → "Credentials".
1. Click "Rotate encryption key".
1. In the "Old encryption key" field, enter the current key from the DDP configuration (`security.secretKey`).
1. In the "New encryption key" field, enter the new key. It must meet the same requirements as during initial setup:
   - The key must be 16, 24, or 32 bytes long.
   - The key may contain only printable ASCII characters.
   - The key must not consist of a single repeated character.
1. Click "Re-encrypt".
1. Wait for the operation to finish and check the results table. For each category, the table shows the number of successfully re-encrypted, skipped, and failed records.
1. Set `security.secretKey` in the DDP Backend configuration to the new key.
1. Restart DDP Backend.

{{< alert level="info" >}}
After re-encryption, credentials remain unavailable until you specify the new key in the backend configuration and restart the service.
{{< /alert >}}

## Operation results

The results table shows three counters for each data category:

- "Successful" — records re-encrypted with the new key.
- "Skipped" — records already accessible with the new key, for example, records re-encrypted earlier.
- "Failed" — records that could not be re-encrypted, usually because the old key is incorrect or the data is corrupted.

The "Total" row shows totals across all categories.

If the "Failed" column contains non-zero values, do not update the backend configuration until you resolve the errors. Verify the old key and repeat the operation.
