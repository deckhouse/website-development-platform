# DefectDojo

## Содержание

- CreateDefectdojoEngagement
- CreateDefectdojoProduct
- DeleteDefectdojoProduct

## CreateDefectdojoEngagement

CreateDefectdojoEngagement — создаёт новый engagement в системе DefectDojo. Действие использует DefectDojo API v2.

### Пример запроса

```yaml
name: example engagement
product: '1'
target_start: '2024-06-01'
target_end: '2024-06-30'
lead: '1'
```

### Спецификация запроса

Список полей соответствует официальному API DefectDojo, `/api/v2/engagements`, подробнее — [в документации Defectdojo](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).

### Учётные данные

* `token` — API v2 Key пользователя, от имени которого будет запускаться выполнение действия.

## CreateDefectdojoProduct

CreateDefectdojoProduct — создаёт новый продукт в системе DefectDojo. Действие использует DefectDojo API v2.

### Пример запроса

```yaml
name: example
description: example description
prod_type: 1
```

### Спецификация запроса

Список полей соответствует официальному API DefectDojo, `/api/v2/products`, подробнее — [в документации Defectdojo](https://demo.defectdojo.org/api/v2/oa3/swagger-ui/).

### Учётные данные

* `token` — API v2 Key пользователя, от имени которого будет запускаться выполнение действия.

## DeleteDefectdojoProduct

DeleteDefectdojoProduct — удаляет продукт из DefectDojo. Действие использует DefectDojo API v2.

### Пример запроса

```yaml
id: 1
```

### Спецификация запроса

| Название    | Обязательность | Описание                                           | Значение по умолчанию  |
|-------------|----------------|----------------------------------------------------|------------------------|
| id          | Да             | Идентификатор продукта, который необходимо удалить | -                      |

### Учётные данные

* `token` — API v2 Key пользователя, от имени которого будет запускаться выполнение действия.
