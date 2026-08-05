---
title: DeleteNexusRole
weight: 80
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

DeleteNexusRole — deletes a role from Nexus Repository Manager 3.

### Request example

```yaml
id: example-role
```

### Request specification

| Field  | Required          | Description                                    |
| ------ | ----------------- | --------------------------------------------- |
| id     | Yes                | Identifier of the role to delete                |
