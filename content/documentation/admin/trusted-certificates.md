---
title: Trusted certificates
menuTitle: Trusted certificates
description: Add certificates for secure HTTPS connections from Deckhouse Development Platform to external services.
---

Trusted certificates allow you to upload root and intermediate certificate authority (CA) certificates or server certificates to Deckhouse Development Platform (DDP). The platform uses them for TLS/SSL verification when connecting to external services over HTTPS, for example when accessing external service APIs from data sources and widgets.

This mechanism provides secure connections to services that use self-signed or corporate certificates without disabling SSL verification.

{{< alert level="info" >}}
Add certificates for Dex, PostgreSQL, and Redis through the module configuration. DDP connects to these internal services before enabling the trusted certificate mechanism.
{{< /alert >}}

Configure trusted certificates under "Administration" → "Trusted certificates".

## Configuration

Specify the following parameters when adding a trusted certificate:

| Parameter | Description |
|-----------|-------------|
| "Name" | Certificate name displayed in the interface, for example "Corporate CA" |
| "Certificate (PEM)" | Certificate content in PEM format. Required when creating the certificate |

Specify the certificate in PEM format:

```sh
-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJAKL...
...
-----END CERTIFICATE-----
```

After you save the certificate, its card displays data extracted from the PEM content, including the subject, issuer, validity period, and alternative names.

When editing a certificate, the "Certificate (PEM)" field is hidden. For security, the API does not return the saved value. To update a certificate, create a new record and delete the old one if necessary.

## Usage

Uploaded trusted certificates are automatically added to the root certificate pool used by DDP Backend for outgoing HTTPS requests. This pool is used for:

- requests to external services from actions, widgets, data sources, and status checks;
- requests to infrastructure system APIs such as GitLab, Kubernetes, Vault, and Prometheus.

If an external service uses a certificate issued by your own or a corporate CA, add the corresponding root or intermediate certificate under "Trusted certificates". The platform then trusts the connection without disabling SSL verification.

{{< alert level="info" >}}
Instead of disabling SSL verification for an external service with the "Disable SSL verification" option, add the required certificates as trusted. This preserves authentication and connection security.
{{< /alert >}}

Changes to the trusted certificate list, including additions, updates, and deletions, apply to new outgoing connections without restarting DDP.

## Access permissions

Trusted certificate management is controlled by the following global permissions:

- `read:trusted-certificates` — view the certificate list and details;
- `create:trusted-certificates` — add certificates;
- `update:trusted-certificates` — edit the name of an existing certificate;
- `delete:trusted-certificates` — delete certificates.

For details about the role model, refer to the [RBAC documentation](../security/rbac/).
