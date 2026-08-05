---
title: AssignNexusRole
weight: 70
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

AssignNexusRole — assigns roles to an existing user in Nexus Repository Manager 3. The action retrieves the user's current configuration and merges the user's existing roles with the new ones.

### Request example

```yaml
userId: example-user
roles:
  - example-role
  - another-role
```

### Request specification

| Field    | Required          | Description                                                             |
| -------- | ----------------- | ----------------------------------------------------------------------- |
| userId   | Yes                | Identifier of the user to which roles are assigned                        |
| roles    | Yes                | List of role identifiers to assign to the user                            |

### How it works

1. Retrieves the user's current configuration from Nexus.
1. Merges the user's existing roles with the new roles from the request.
1. Updates the user with the merged list of roles.

### Note

The user must exist in Nexus before roles can be assigned. If the user is not found, the action fails with an error. All specified roles must also exist in Nexus.
