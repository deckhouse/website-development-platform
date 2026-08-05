---
title: CreateDefectdojoProduct
weight: 10
---


{{< alert level="info" >}}
Running this action requires a token — an API v2 Key for the user on whose behalf the action will be run.
{{< /alert >}}

CreateDefectdojoProduct — creates a new product in DefectDojo. The action uses the DefectDojo API v2.

### Request example

```yaml
name: example
description: example description
prod_type: 1
```

### Request specification

The list of fields corresponds to the official DefectDojo API, `/api/v2/products`, for more information see [the DefectDojo documentation](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).
