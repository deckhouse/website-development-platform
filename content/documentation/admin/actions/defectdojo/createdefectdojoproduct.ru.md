---
title: CreateDefectdojoProduct
weight: 10
---

{{< alert level="info" >}}
Для выполнения действия требуется токен API v2 Key пользователя, от имени которого выполняется действие.
{{< /alert >}}

CreateDefectdojoProduct — создаёт новый продукт в системе DefectDojo. Действие использует DefectDojo API v2.

## Пример запроса

Пример тела запроса действия:

```yaml
name: example
description: example description
prod_type: 1
```

## Спецификация запроса

Список полей соответствует официальному API DefectDojo, `/api/v2/products`, подробнее — [в документации DefectDojo](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).
