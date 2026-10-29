---
title: DeleteNexusUser
weight: 100
---

{{< alert level="info" >}}
Running this action requires a token: the `admin:password` string encoded in Base64.
The token is used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

DeleteNexusUser — deletes a user from Nexus Repository Manager 3.

## Request example

Example of the action request body:

```yaml
userId: example-user
```

## Request specification

The action request body supports the following parameters:

| Name     | Required          | Description                                            |
| -------- | ----------------- | ------------------------------------------------------ |
| `userId` | Yes                | Identifier of the user to delete                         |
