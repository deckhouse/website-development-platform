---
title: DeleteCodeScoringProject
weight: 20
---

{{< alert level="info" >}}
This action does not use the DDP credentials mechanism. If CodeScoring requires authentication, add the required HTTP header, such as `Authorization`, in the [URL and HTTP headers](../overview/#url-and-http-headers) settings. DDP passes this header in the request to CodeScoring.
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
