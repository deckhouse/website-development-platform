---
title: Credentials
menuTitle: Credentials
d8Edition: ee
moduleStatus: experimental
---

Credentials are a mechanism for securely storing and using access credentials to external services (tokens, passwords, API keys, etc.) in the DDP platform.

## How it works

All interactions with infrastructure services in DDP occur using the credentials of a specific user. Credentials are encrypted before being stored in the database and are decrypted only when needed in actions, data sources, and widgets.

For more information on how they work, encryption, and configuration, see the [documentation](../../admin/security/credentials/).

## Filling in credentials

The platform administrator creates credential types in the "Administration" → "Credentials" section. For each credential type, you can specify your personal details in the "Credentials" section of your profile.

For example, if an administrator created the **Kubernetes token** credential type, you can add a personal token for Kubernetes access in your profile.

## Working with Credentials

- After saving credentials, you cannot view their value; you can only update it.
- Your profile displays information about whether certain credentials are filled in.
- Credentials are used automatically in actions, data sources, and widgets where they are specified in the configuration.

The approach to using credentials in actions, data sources, and widgets is described in the relevant sections of the documentation.
