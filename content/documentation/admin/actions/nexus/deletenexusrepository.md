---
title: DeleteNexusRepository
weight: 20
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

`DeleteNexusRepository` — deletes an existing repository from Nexus Repository Manager 3.

### Request example

```yaml
name: my-repo-to-delete
```

### Request specification

| Field  | Required          | Description                                       |
| ------ | ----------------- | ------------------------------------------------ |
| `name` | Yes                | Name of the repository to delete                    |

### Algorithm

- Sends a `DELETE` request to `/service/rest/v1/repositories/{name}`, where `{name}` is the value of the `name` field.
- If the repository is found and deleted, the request returns status code 204.
- If the repository is not found, the request returns a 404 error.
