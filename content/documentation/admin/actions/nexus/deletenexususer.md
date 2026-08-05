---
title: DeleteNexusUser
weight: 100
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

DeleteNexusUser — deletes a user from Nexus Repository Manager 3.

### Request example

```yaml
userId: example-user
```

### Request specification

| Field    | Required          | Description                                            |
| -------- | ----------------- | ------------------------------------------------------ |
| userId   | Yes                | Identifier of the user to delete                         |
