---
title: AssignNexusRole
weight: 70
---

{{< alert level="info" >}}
Running this action requires a token: the `admin:password` string encoded in Base64.
The token is used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

AssignNexusRole — assigns roles to an existing user in Nexus Repository Manager 3.
The action retrieves the user's current configuration and merges the user's existing roles with the new ones.

## Request example

Example of the action request body:

```yaml
userId: example-user
roles:
  - example-role
  - another-role
```

## Request specification

The action request body supports the following parameters:

| Name     | Required          | Description                                                             |
| -------- | ----------------- | ----------------------------------------------------------------------- |
| `userId` | Yes                | Identifier of the user to which roles are assigned                        |
| `roles`  | Yes                | List of role identifiers to assign to the user                            |

## How it works

The action runs in the following order:

1. Retrieves the user's current configuration from Nexus.
1. Merges the user's existing roles with the new roles from the request.
1. Updates the user with the merged list of roles.

## Requirements

The user must exist in Nexus before roles can be assigned.
If the user is not found, the action fails with an error.
All specified roles must also exist in Nexus.
