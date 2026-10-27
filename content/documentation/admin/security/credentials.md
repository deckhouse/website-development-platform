---
title: Credentials
weight: 30
---

Credentials let you securely store and use access details for external services in actions, datasources, widgets, and other Deckhouse Development Portal (DDP, portal) components.
All credentials are encrypted before being saved to the database and decrypted only when used.

## How it works

Credentials are stored in the portal in encrypted form and substituted into requests at run time.

### Credential types

The portal administrator creates credential types in the "Administration" → "Credentials" section. Each type has the following fields:

- "Name": Display name of the type.
- "Identifier": Unique identifier used in configurations.
- "Description": Hint for users.

### User credentials

Users fill in their personal credentials in the "Profile" → "Credentials" section.
For each credential type, a user can specify their own value (for example, an access token, password, or API key).

After saving, a user cannot view the credential value, only update it.
The profile shows whether each credential is filled in.

### Selecting storage for a credential type

When you create or edit a credential type, you specify the storage.
By default, "Database" is used: values are stored in PostgreSQL.
If you select Vault, values are stored in HashiCorp Vault or Deckhouse Stronghold.
This option is available only if the Vault integration is enabled and configured in the portal.

A credential type always uses one selected storage.
You can change the storage of an existing type; however, credentials already saved in the old storage are not moved automatically.

### Storing in the database

Credentials with the "Database" storage type are saved in PostgreSQL.
Before saving, values are encrypted using the AES-GCM algorithm.

Encryption parameters:

- Algorithm: AES-GCM.
- Key size: 16, 24, or 32 bytes (128, 192, or 256 bits).
- Nonce size: 12 bytes.
- Encryption key: The `security.secretKey` module parameter. For details, see [Encryption key](#encryption-key).

Encrypted values are saved in the database with the `GCM:` prefix, which lets the system identify the encryption method during decryption.

{{< alert level="warning" >}}
If you change the encryption key (`security.secretKey`) in the module settings without re-encrypting first, all current credentials become unavailable.
Use [encryption key rotation](../encryption-key-rotation/) in the web interface.
{{< /alert >}}

### Storing in HashiCorp Vault

If the Vault storage is selected for a credential type, the portal saves user credentials in HashiCorp Vault or Deckhouse Stronghold.

How it works:

- Vault uses the KV Secrets Engine v2.
  The secrets path is set in the Vault settings in DDP (for example, `secrets/data/ddp`).
- A separate secret is created for each user at the `{path}/credentials/{type_uuid}/users/{user_uuid}` path.
  The secret contains a single `value` key whose value is the encrypted credentials.
  Encryption uses the DDP key (`security.secretKey`).
- The connection to Vault uses `AppRole` (AppRole Role ID and AppRole Secret ID).
  AppRole Role ID and AppRole Secret ID are stored in the DDP database in encrypted form.
  The Vault token is renewed automatically; if renewal fails, the portal signs in again using AppRole.

To configure Vault in DDP, follow these steps:

1. In the "Administration" → "Credentials" section, click "Vault settings".
1. Enable Vault ("Enabled").
1. Fill in the parameters:
   - `URL`: Vault server address (for example, `https://vault.example.com`).
   - "AppRole Role ID": AppRole ID obtained when creating the role in Vault.
   - "AppRole Secret ID": AppRole secret, used together with Role ID for authentication in Vault.
   - "Path": Path to secrets in KV v2.
     For KV v2, the path must contain the `/data/` segment (for example, `secrets/data/ddp` or `secrets/data/ddp/credentials`).
     The mount (for example, `secrets`) must be created in Vault in advance; the subdirectories after `data/` are created by the portal automatically when credentials are saved.
1. Click "Test connection" to make sure DDP can authenticate and access Vault.
1. Save the configuration.

After the configuration is saved, the Vault option becomes available in the storage list when you create or edit a credential type.
If Vault is disabled or not configured, types with Vault storage remain in the list, but credentials of these types cannot be filled in or used until Vault is enabled and configured.

#### Access permissions

To view and change the Vault configuration and test the connection, the global permission to edit credential types is required: `edit:user-access-credentials-types`.

#### Configuration specifics

When configuring the Vault integration, consider the following:

- Only KV Secrets Engine v2 is supported.
  The path in the settings must include `/data/` (for example, `mount/data/subdirectory`).
- There is one Vault configuration per DDP instance: one URL, one AppRole, and one base path.
  All credential types with Vault storage use this path (subdirectories are created automatically based on the type UUID).
- You can clear all saved values of a type from the current storage (database or Vault) at any time with the "Clear storage" action in the credential types table.
- Changing the DDP encryption key (`security.secretKey`) without re-encrypting first also makes credentials in Vault unreadable, because values in Vault are stored encrypted.
  Use [encryption key rotation](../encryption-key-rotation/).

### Using credentials

Credentials are decrypted only when they need to be used, immediately before accessing an external service.
After use, decrypted values are not kept in memory longer than necessary.

## Integration with portal components

Credentials are used by actions, datasources, widgets, and status checks.

### Actions

In the action configuration, you can specify the list of required credentials in the "Credentials" field.
For each entry in the list, you set an "Identifier" that matches the credential type ID and a "Credential type" selected from the list.

When running an action, the system does the following:

1. Determines the user on whose behalf the action runs (the initiator, or the account selected in the "Account for execution" field if the "Select account for execution" option is enabled).
1. Retrieves this user's credentials from the database.
1. Decrypts the credential values.
1. Substitutes the decrypted values into the action configuration templates (URL, headers, request body) using placeholders like `{{.credentials.identifier}}`.

### Datasources

In the datasource configuration, you can specify the list of required credentials in the same way as for actions.
When synchronizing data, the system does the following:

1. Uses the credentials of the system user specified in the datasource configuration (the "System account" field).
1. Retrieves and decrypts the system user's credentials.
1. Substitutes the values into the datasource request templates.

{{< alert level="warning" >}}
Datasources always use the credentials of the system user, not of the user who started the synchronization.
{{< /alert >}}

### Widgets

Widgets can use credentials to get data from external services.
The mechanism is similar to actions:

1. The user is determined (the request initiator, or the account selected in the "Account for widget" field if the "Select account for widget" option is enabled).
1. The credentials are retrieved and decrypted.
1. The values are substituted into the widget configuration.

### Status checks

Status checks use credentials to check the state of entities.
In the status check rule configuration, you can specify the list of required credentials.
When running a check, the system does the following:

1. Uses the credentials of the system user specified in the status check rule configuration (the "System account" field).
1. Retrieves and decrypts the system user's credentials.
1. Substitutes the values into the rule configuration templates (URL, headers, request parameters) using placeholders like `{{.credentials.identifier}}`.

{{< alert level="warning" >}}
Status checks always use the credentials of the system user specified in the rule configuration.
If no system user is specified, the rule is skipped during the check.
{{< /alert >}}

### External services

External services can have their own list of credentials.
If an action, datasource, or widget uses an external service (the "Use external service" option is enabled), then:

1. The component inherits the credentials from the external service configuration if its own configuration does not specify credentials.
1. Credentials specified directly in the component configuration take precedence.

## Configuration

To work with credentials, configure the encryption key.

### Encryption key

The encryption key is set in the `security.secretKey` parameter of the `development-platform` module: in the `development-platform` ModuleConfig (`spec.settings.security.secretKey`) or in the module settings in the Deckhouse web UI.

Example of a ModuleConfig fragment:

```yaml
apiVersion: deckhouse.io/v1alpha1
kind: ModuleConfig
metadata:
  name: development-platform
spec:
  settings:
    security:
      secretKey: <SECRET_KEY>
```

Key requirements:

- Length: 16, 24, or 32 printable ASCII characters (128, 192, or 256 bits).
- The key must not consist of a single repeated character.
- The key must not match common values, such as `1234567890123456` or `passwordpassword`.

If the key does not meet the requirements, DDP Backend does not start.

### Vault integration

The connection to HashiCorp Vault (URL, AppRole, KV v2 path) is configured in the web interface in the "Administration" → "Credentials" → "Vault settings" section.
The module settings define only the encryption key (`security.secretKey`), which is also used to encrypt values stored in Vault.

## Security

Credentials are protected as follows:

- Credentials are never sent to the user in decrypted form.
- Decryption happens only immediately before a request is sent.
- Decrypted values are not logged.
