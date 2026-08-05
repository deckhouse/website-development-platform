---
title: CreateNexusRole
weight: 60
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

CreateNexusRole — creates a new role in Nexus Repository Manager 3. Roles combine privileges and can include other roles.

### Request example

```yaml
id: example-role
name: Example Role
description: Example role description
privileges:
  - nx-repository-view-*-*-read
  - nx-repository-view-maven2-*-browse
roles: []
```

### Request specification

| Field         | Required          | Description                                                                     |
| ------------- | ----------------- | ------------------------------------------------------------------------------ |
| id            | Yes                | Unique role identifier                                                            |
| name          | Yes                | Role name                                                                         |
| description   | No                 | Role description                                                                  |
| privileges    | No                 | List of privilege names assigned to the role                                      |
| roles         | No                 | List of other role identifiers included in this role                             |
