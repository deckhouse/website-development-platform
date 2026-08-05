---
title: DeleteNexusPrivilege
weight: 50
---


{{< alert level="info" >}}
Running this action requires a token — a base64(`admin:password`) string used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

DeleteNexusPrivilege — deletes a privilege from Nexus Repository Manager 3.

### Request example

```yaml
name: example-privilege
```

### Request specification

| Field  | Required          | Description                                     |
| ------ | ----------------- | ---------------------------------------------- |
| name   | Yes                | Name of the privilege to delete                  |
