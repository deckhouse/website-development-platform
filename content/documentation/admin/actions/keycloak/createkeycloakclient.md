---
title: CreateKeycloakClient
weight: 10
---


{{< alert level="info" >}}
This action requires the following credentials:
* `username` — the username under which the action runs.
* `password` — the password for that user.
{{< /alert >}}

CreateKeycloakClient — creates a new client in Keycloak.

### Request example

```yaml
realm: master
config:
  clientId: example
  name: example
  enabled: true
  clientAuthenticatorType: client-secret
  secret: secret
  defaultClientScopes:
  - roles
  - profile
  - email
  optionalClientScopes:
  - address
  - phone
  - offline_access
```

### Request specification

| Name       | Required | Description                                                                                                                                                                       | Possible values                                                                                     |
| ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| realm      | Yes      | Keycloak realm in which to create the client                                                                                                                                        | -                                                                                                   |
| config     | Yes      | Parameters for the client to create, as defined in the [Keycloak ClientRepresentation specification](https://www.keycloak.org/docs-api/latest/rest-api/index.html#ClientRepresentation) | -                                                                                                   |
