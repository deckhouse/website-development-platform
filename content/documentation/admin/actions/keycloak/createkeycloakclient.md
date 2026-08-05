---
title: CreateKeycloakClient
weight: 10
---


{{< alert level="info" >}}
Running this action requires credentials:
* `username` — the username on whose behalf the action will be run.
* `password` — the password of the user on whose behalf the action will be run.
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
| realm      | Yes      | Realm in Keycloak in which to create the client                                                                                                                                    | -                                                                                                   |
| config     | Yes      | Parameters of the client to create, according to the [Keycloak ClientRepresentation specification](https://www.keycloak.org/docs-api/latest/rest-api/index.html#ClientRepresentation) | -                                                                                                   |
