---
title: DeleteNexusPrivilege
weight: 50
---

{{< alert level="info" >}}
Running this action requires a token: the `admin:password` string encoded in Base64.
The token is used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

DeleteNexusPrivilege — deletes a privilege from Nexus Repository Manager 3.

## Request example

Example of the action request body:

```yaml
name: example-privilege
```

## Request specification

The action request body supports the following parameters:

| Name   | Required          | Description                                     |
| ------ | ----------------- | ---------------------------------------------- |
| `name` | Yes                | Name of the privilege to delete                  |
