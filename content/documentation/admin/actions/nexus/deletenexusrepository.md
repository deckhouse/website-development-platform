---
title: DeleteNexusRepository
weight: 20
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used as Basic Auth for requests to Nexus.
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

- Performs a `DELETE` request to `/service/rest/v1/repositories/{name}`, where `{name}` is the value of the `name` field.
- If the repository is found and deleted, a 204 is returned.
- If it is not found, a 404 error is returned.
