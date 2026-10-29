---
title: DeleteNexusRepository
weight: 20
---

{{< alert level="info" >}}
Running this action requires a token: the `admin:password` string encoded in Base64.
The token is used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

DeleteNexusRepository — deletes an existing repository from Nexus Repository Manager 3.

## Request example

Example of the action request body:

```yaml
name: my-repo-to-delete
```

## Request specification

The action request body supports the following parameters:

| Name   | Required          | Description                                       |
| ------ | ----------------- | ------------------------------------------------ |
| `name` | Yes                | Name of the repository to delete                    |

## How it works

The action runs in the following order:

- Sends a `DELETE` request to `/service/rest/v1/repositories/{name}`, where `{name}` is the value of the `name` field.
- If the repository is found and deleted, the request returns status code 204.
- If the repository is not found, the request returns a 404 error.
