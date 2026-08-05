---
title: CreateDefectdojoProduct
weight: 10
---


{{< alert level="info" >}}
This action requires an API v2 key for the user on whose behalf it will run.
{{< /alert >}}

CreateDefectdojoProduct — creates a new product in DefectDojo. The action uses the DefectDojo API v2.

### Request example

```yaml
name: example
description: example description
prod_type: 1
```

### Request specification

The fields correspond to the official DefectDojo API endpoint `/api/v2/products`. For details, see the [DefectDojo documentation](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).
