---
title: External services
menuTitle: External services
description: Configure authentication for external infrastructure services used by platform objects.
---

External services configure authentication for external infrastructure systems such as GitLab, Kubernetes, and DefectDojo.

Configure external services under "Administration" → "External services".

## Configuration

An external service has the following parameters:

| Parameter | Description |
|-----------|-------------|
| "Name" | External service name displayed in the interface |
| "Identifier" | Unique human-readable identifier (slug) |
| "Owner" | User responsible for the external service |
| "Owner team" | Team that owns the external service |
| "URL" | Base URL for requests to the external service |
| "Credentials" | Available credentials, such as tokens, that can be included in requests |
| "Headers" | HTTP headers automatically added to requests |
| "Disable SSL verification" | Disables verification of the external service's SSL certificate, for example when using self-signed certificates. Instead, add the required certificates to [trusted certificates](../trusted-certificates/) |
| "System credential" | Credential used for scheduled jobs such as data source synchronization and status checks |

## Using external services

External services can be connected to the following platform objects:

- actions;
- widgets;
- data sources;
- entity status checks.

Connect a service on the "Authorization" tab in the settings of the corresponding object.

Each object can explicitly override parameters configured for the external service. For example, if an external service contains an authentication token but an action requires different credentials, specify alternative values. Only the specified parameter is overridden; all other values come from the external service configuration.

## Object-specific behavior

### Actions

- Multiple external services can be connected to one action.
- One service can be selected as the default.
- The "System credential" parameter is not used for actions.
- When running an action, you can select the external service to use. If you do not select a service, the default service is used. If no default is configured, the first external service in the list is used.

### Widgets

- Multiple external services can be connected to one widget.
- One service can be selected as the default.
- A future release will allow users to change the service while viewing a widget.
- The "System credential" parameter is not used for widgets.

### Data sources

- Only one external service can be connected.
- If the service has a system credential, it is used by default during synchronization.

### Status checks

- One external service can be assigned to a status check.
- If the service has a system credential, it is used for automatic checks.

## Authorization headers for external services

When configuring an external service, specify the HTTP headers required for authentication. The following sections list supported external services, authentication methods, and required headers.

{{< alert level="info" >}}
The examples below show possible authentication methods for each service. Some services may support other methods. The platform supports any method that can be passed through HTTP headers.
{{< /alert >}}

### CodeScoring

Authentication type: API token.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `<token>` |

Example:

```sh
Authorization: <your-api-token>
```

### ClickHouse

Authentication type: Basic Authentication or the `X-ClickHouse-User` and `X-ClickHouse-Key` headers.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Basic <base64-encoded-credentials>` |
| `X-ClickHouse-User` | `<username>` |
| `X-ClickHouse-Key` | `<password>` |

Basic Authentication example:

1. Create the `username:password` string.
1. Encode it in Base64: `echo -n "username:password" | base64`.
1. Add the header:

```sh
Authorization: Basic <base64-encoded-credentials>
```

Example using ClickHouse headers:

```sh
X-ClickHouse-User: <username>
X-ClickHouse-Key: <password>
```

### DefectDojo

Authentication type: API v2 key token.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Token <token>` |

Example:

```sh
Authorization: Token <your-defectdojo-api-v2-key>
```

### Bitbucket

Authentication type: bearer token (personal access token).

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Bearer <token>` |

Example:

```sh
Authorization: Bearer <your-bitbucket-personal-access-token>
```

### Docker Registry

Authentication type: Basic Authentication.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Basic <base64-encoded-credentials>` |

Example:

1. Create the `username:password` string.
1. Encode it in Base64: `echo -n "username:password" | base64`.
1. Add the header:

```sh
Authorization: Basic <base64-encoded-credentials>
```

### GitLab

Authentication type: personal access token or project access token.

Headers:

| Header | Value format |
|--------|--------------|
| `Private-Token` | `<token>` |

Example:

```sh
Private-Token: <your-gitlab-token>
```

For instructions on creating a GitLab token, refer to the [GitLab authentication documentation](https://docs.gitlab.com/api/rest/authentication/).

### GitHub

Authentication type: personal access token.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Bearer <token>` |

Example:

```sh
Authorization: Bearer <your-github-token>
```

Create the token in GitHub under "Settings" → "Developer settings" → "Personal access tokens".

### Harbor

Authentication type: Basic Authentication.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Basic <base64-encoded-credentials>` |

Example:

1. Create the `username:password` string.
1. Encode it in Base64: `echo -n "username:password" | base64`.
1. Add the header:

```sh
Authorization: Basic <base64-encoded-credentials>
```

### Jenkins

Authentication type: Basic Authentication (username and password).

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Basic <base64-encoded-credentials>` |

Example:

1. Create the `username:password` string, where:
   - `username` is the Jenkins username.
   - `password` is the user's password.
1. Encode the string in Base64: `echo -n "username:password" | base64`.
1. Add the header:

```sh
Authorization: Basic <base64-encoded-credentials>
```

### Jira

Authentication type: Basic Authentication.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Basic <base64-encoded-credentials>` |

Example:

1. Create the `username:password` string.
1. Encode it in Base64: `echo -n "username:password" | base64`.
1. Add the header:

```sh
Authorization: Basic <base64-encoded-credentials>
```

### Kaiten

Authentication type: API token.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Bearer <token>` |

Example:

```sh
Authorization: Bearer <your-kaiten-api-token>
```

### Kubernetes

Authentication type: bearer token (service account token or user token).

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Bearer <token>` |

Example:

```sh
Authorization: Bearer <your-kubernetes-token>
```

### Nexus

Authentication type: Basic Authentication.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Basic <base64-encoded-credentials>` |

Example:

1. Create the `username:password` string.
1. Encode it in Base64: `echo -n "username:password" | base64`.
1. Add the header:

```sh
Authorization: Basic <base64-encoded-credentials>
```

### OpenSearch

Authentication type: Basic Authentication.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Basic <base64-encoded-credentials>` |

Example:

1. Create the `username:password` string.
1. Encode it in Base64: `echo -n "username:password" | base64`.
1. Add the header:

```sh
Authorization: Basic <base64-encoded-credentials>
```

### Prometheus

Authentication type: bearer token or Basic Authentication.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Bearer <token>` or `Basic <base64-encoded-credentials>` |

Bearer token example:

```sh
Authorization: Bearer <your-token>
```

Basic Authentication example:

```sh
Authorization: Basic <base64-encoded-credentials>
```

### SonarQube

Authentication type: bearer token.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Bearer <token>` |

Example:

```sh
Authorization: Bearer <your-token>
```

### Svacer

Authentication type: bearer token.

Headers:

| Header | Value format |
|--------|--------------|
| `Authorization` | `Bearer <token>` |

Example:

```sh
Authorization: Bearer <your-token>
```

### Vault

Authentication type: token authentication.

Headers:

| Header | Value format |
|--------|--------------|
| `X-Vault-Token` | `<token>` |

Example:

```sh
X-Vault-Token: <your-vault-token>
```
