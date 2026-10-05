---
title: Encryption key rotation
description: Safely re-encrypt stored credentials and replace the DDP Backend encryption key.
weight: 35
---

Deckhouse Development Portal (DDP) lets you securely replace the encryption key (`security.secretKey`) without losing credentials or other encrypted values. The operation runs from the web interface and re-encrypts all data stored with the current key.

## Reasons to rotate the key

Rotate the encryption key when you need to replace the `security.secretKey` module parameter, for example, to comply with a security policy or after the key has been compromised.

The parameter is set in the `development-platform` ModuleConfig (`spec.settings.security.secretKey`) or in the module settings in the Deckhouse Platform web interface.

{{< alert level="warning" >}}
Do not change `security.secretKey` in the module settings before re-encryption finishes in the web interface. Otherwise, the stored values will become unavailable.
{{< /alert >}}

## Access permissions

Key rotation requires the global `rotate:encryption-key` permission.

## Data that is re-encrypted

The operation affects the following data categories:

- "User credentials" — values in PostgreSQL for credential types that use the "Database" storage.
- "AI provider credentials" — personal credentials for AI integrations.
- "Action temporary responses" — encrypted temporary responses in action execution records.
- "Vault secrets" — credential values stored in HashiCorp Vault or Deckhouse Stronghold.
- "Vault configuration" — the AppRole Role ID and AppRole Secret ID in the Vault integration settings.
- Template registry connection passwords. The results table has no separate row for them: they are counted in the "Total" row.

## Procedure

1. Back up the DDP PostgreSQL database. If re-encryption fails, you can restore the encrypted values from the backup.
1. Go to "Administration" → "Credentials".
1. Click "Rotate encryption key".
1. In the "Old encryption key" field, enter the current key (the `security.secretKey` module parameter).
1. In the "New encryption key" field, enter the new key. It must meet the same requirements as during initial setup:
   - The key must be 16, 24, or 32 printable ASCII characters long.
   - The key must not consist of a single repeated character.
   - The key must not be a commonly used value, such as `1234567890123456` or `passwordpassword`.
   - The new key must differ from the old one.
1. Click "Re-encrypt".
1. Wait for the operation to finish and check the results table. For each category, the table shows the number of successfully re-encrypted, skipped, and failed records.
1. Set the new key in the `security.secretKey` module parameter: in the `development-platform` ModuleConfig (`spec.settings.security.secretKey`) or in the module settings in the Deckhouse Platform web interface.

   Example of a ModuleConfig fragment:

   ```yaml
   apiVersion: deckhouse.io/v1alpha1
   kind: ModuleConfig
   metadata:
     name: development-platform
   spec:
     settings:
       security:
         secretKey: <NEW_SECRET_KEY>
   ```

   After the module settings change, DDP Backend and the DDP workers restart automatically with the new key.

{{< alert level="info" >}}
After re-encryption and until DDP Backend restarts with the new key, the stored values cannot be decrypted.

Values that users save in this interval are encrypted with the old key and cannot be read after the restart. To re-encrypt them, run the rotation again with the same old and new keys: values that are already encrypted with the new key are skipped.
{{< /alert >}}

Browser sessions and impersonation sessions are signed with `security.secretKey`. After the key changes, current sessions become invalid: the web interface tries to renew the session through Dex and redirects the user to the sign-in page if renewal fails. Active impersonation sessions end.

## Operation results

The results table shows three counters for each data category:

- "Succeeded" — records re-encrypted with the new key.
- "Skipped" — records already accessible with the new key, for example, records re-encrypted earlier, and records without a stored value.
- "Failed" — records that could not be re-encrypted, usually because the old key is incorrect or the data is corrupted.

The "Total" row shows totals across all categories.

If the "Failed" column contains non-zero values, do not change the key in the module settings until you resolve the errors. Verify the old key and repeat the operation.
