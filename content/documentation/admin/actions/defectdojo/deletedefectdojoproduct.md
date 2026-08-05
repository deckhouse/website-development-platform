---
title: DeleteDefectdojoProduct
weight: 20
---


{{< alert level="info" >}}
Running this action requires a token — an API v2 Key for the user on whose behalf the action will be run.
{{< /alert >}}

DeleteDefectdojoProduct — deletes a product from DefectDojo. The action uses the DefectDojo API v2.

### Request example

```yaml
id: 1
```

### Request specification

| Name          | Required | Description                                   | Default value             |
| ------------- | -------- | ------------------------------------------------ | ------------------------ |
| id            | Yes      | Identifier of the product to delete               | -                        |
