---
title: CreateDefectdojoEngagement
weight: 30
---


{{< alert level="info" >}}
This action requires an API v2 key for the user on whose behalf it will run.
{{< /alert >}}

CreateDefectdojoEngagement — creates a new engagement in DefectDojo. The action uses the DefectDojo API v2.

### Request example

```yaml
name: example engagement
product: '1'
target_start: '2024-06-01'
target_end: '2024-06-30'
lead: '1'
```

### Request specification

The fields correspond to the official DefectDojo API endpoint `/api/v2/engagements`. For details, see the [DefectDojo documentation](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).
