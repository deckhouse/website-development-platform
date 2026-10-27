---
title: DeleteNexusRole
weight: 80
---

{{< alert level="info" >}}
Running this action requires a token: the `admin:password` string encoded in Base64.
The token is used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

DeleteNexusRole — deletes a role from Nexus Repository Manager 3.

## Request example

Example of the action request body:

```yaml
id: example-role
```

## Request specification

The action request body supports the following parameters:

| Name   | Required          | Description                                    |
| ------ | ----------------- | --------------------------------------------- |
| `id`   | Yes                | Identifier of the role to delete                |
