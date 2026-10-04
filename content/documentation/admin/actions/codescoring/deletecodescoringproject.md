---
title: DeleteCodeScoringProject
weight: 20
---

{{< alert level="info" >}}
The action does not use credentials. Authenticate in CodeScoring through the HTTP headers of the action:
add the authentication header in the [URL and HTTP headers](../../overview/#url-and-http-headers) settings.
{{< /alert >}}

DeleteCodeScoringProject — deletes a project in CodeScoring by its ID.

### Request example

```yaml
id: 1
```

### Request specification

| Name       | Required | Description                |
| ---------- | -------- | --------------------------- |
| id         | Yes      | Project ID in CodeScoring   |
