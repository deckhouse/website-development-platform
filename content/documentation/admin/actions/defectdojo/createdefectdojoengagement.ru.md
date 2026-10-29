---
title: CreateDefectdojoEngagement
weight: 30
---

{{< alert level="info" >}}
Для выполнения действия требуется токен API v2 Key пользователя, от имени которого выполняется действие.
{{< /alert >}}

CreateDefectdojoEngagement — создаёт новый engagement в системе DefectDojo. Действие использует DefectDojo API v2.

## Пример запроса

Пример тела запроса действия:

```yaml
name: example engagement
product: '1'
target_start: '2024-06-01'
target_end: '2024-06-30'
lead: '1'
```

## Спецификация запроса

Список полей соответствует официальному API DefectDojo, `/api/v2/engagements`, подробнее — [в документации DefectDojo](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).
