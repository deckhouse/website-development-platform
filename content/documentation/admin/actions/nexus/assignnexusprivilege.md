---
title: AssignNexusPrivilege
weight: 40
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used as Basic Auth for requests to Nexus.
{{< /alert >}}

AssignNexusPrivilege — assigns privileges to an existing role in Nexus Repository Manager 3. The action retrieves the role's current configuration and merges its existing privileges with the new ones.

### Request example

```yaml
roleId: example-role
privileges:
  - example-privilege
  - another-privilege
```

### Request specification

| Field        | Required          | Description                                                                     |
| ------------ | ----------------- | ------------------------------------------------------------------------------ |
| roleId       | Yes                | Identifier of the role to which privileges are assigned                          |
| privileges   | Yes                | List of privilege names to assign to the role                                    |

### How it works

1. Retrieves the role's current configuration from Nexus.
1. Merges the role's existing privileges with the new privileges from the request.
1. Updates the role with the merged list of privileges.

### Note

The role must exist in Nexus before privileges can be assigned. If the role is not found, the action fails with an error. All specified privileges must also exist in Nexus.
