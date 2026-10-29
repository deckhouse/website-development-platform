---
title: AssignNexusPrivilege
weight: 40
---

{{< alert level="info" >}}
Running this action requires a token: the `admin:password` string encoded in Base64.
The token is used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

AssignNexusPrivilege — assigns privileges to an existing role in Nexus Repository Manager 3.
The action retrieves the role's current configuration and merges its existing privileges with the new ones.

## Request example

Example of the action request body:

```yaml
roleId: example-role
privileges:
  - example-privilege
  - another-privilege
```

## Request specification

The action request body supports the following parameters:

| Name         | Required          | Description                                                                     |
| ------------ | ----------------- | ------------------------------------------------------------------------------ |
| `roleId`     | Yes                | Identifier of the role to which privileges are assigned                          |
| `privileges` | Yes                | List of privilege names to assign to the role                                    |

## How it works

The action runs in the following order:

1. Retrieves the role's current configuration from Nexus.
1. Merges the role's existing privileges with the new privileges from the request.
1. Updates the role with the merged list of privileges.

## Requirements

The role must exist in Nexus before privileges can be assigned.
If the role is not found, the action fails with an error.
All specified privileges must also exist in Nexus.
