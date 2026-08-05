---
title: CreateNexusPrivilege
weight: 30
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

CreateNexusPrivilege — creates a new privilege in Nexus Repository Manager 3. Privileges define access rights to repositories and other Nexus resources.

### Request example (repository-view)

```yaml
name: example-privilege
description: Example privilege description
type: repository-view
actions:
  - READ
  - BROWSE
format: maven2
repository: maven-releases
```

### Request example (repository-content-selector)

```yaml
name: content-selector-privilege
description: Privilege with content selector
type: repository-content-selector
actions:
  - READ
format: maven2
repository: maven-releases
contentSelector: my-content-selector
```

### Request example (wildcard)

```yaml
name: wildcard-privilege
description: Wildcard privilege
type: wildcard
pattern: nx-*
actions:
  - READ
```

### Request specification

| Field             | Required           | Description                                                                                                                                                          |
| ----------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name              | Yes                 | Name of the privilege to create. Must be unique within Nexus                                                                                                          |
| description       | No                  | Privilege description                                                                                                                                                 |
| type              | Yes                 | Privilege type: `repository-view`, `repository-content-selector`, `repository-admin`, `application`, `wildcard`                                                      |
| actions           | No                  | List of actions allowed by the privilege (e.g., `READ`, `BROWSE`, `CREATE`, `UPDATE`, `DELETE`)                                                                       |
| format            | No                  | Repository format (e.g., `maven2`, `docker`, `npm`). Used for the `repository-view`, `repository-content-selector`, `repository-admin` types                        |
| repository        | No                  | Repository name. Used for the `repository-view`, `repository-content-selector`, `repository-admin` types                                                            |
| contentSelector   | No                  | Content selector name. Required for the `repository-content-selector` type. If not specified or invalid, the type is automatically converted to `repository-view`   |
| pattern           | No                  | Pattern for the `wildcard` type                                                                                                                                       |
| domain            | No                  | Domain for the `application` type                                                                                                                                     |
| attributes        | No                  | Additional key-value parameters                                                                                                                                       |

### Note

For the `repository-content-selector` type, the content selector must exist in Nexus before the privilege is created. If the content selector is not specified or is invalid, the action automatically converts the privilege type to `repository-view`.
