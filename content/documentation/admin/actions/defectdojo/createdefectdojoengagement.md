---
title: CreateDefectdojoEngagement
weight: 30
---


{{< alert level="info" >}}
Running this action requires a token — an API v2 Key for the user on whose behalf the action will be run.
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

The list of fields corresponds to the official DefectDojo API, `/api/v2/engagements`, for more information see [the DefectDojo documentation](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).
