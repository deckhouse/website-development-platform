---
title: CreateNexusRole
weight: 60
---


{{< alert level="info" >}}
Для выполнения действия необходимо наличие токена — строки base64(`admin:password`), используемой как Basic Auth при запросах к Nexus.
{{< /alert >}}

CreateNexusRole — создаёт новую роль в Nexus Repository Manager 3. Роли объединяют привилегии и могут включать другие роли.

### Пример запроса

```yaml
id: example-role
name: Example Role
description: Example role description
privileges:
  - nx-repository-view-*-*-read
  - nx-repository-view-maven2-*-browse
roles: []
```

### Спецификация запроса

| Поле          | Обязательность    | Описание                                                                       |
| ------------- | ----------------- | ------------------------------------------------------------------------------ |
| id            | Да                | Уникальный идентификатор роли                                                  |
| name          | Да                | Название роли                                                                  |
| description   | Нет               | Описание роли                                                                  |
| privileges    | Нет               | Список названий привилегий, которые назначаются роли                           |
| roles         | Нет               | Список идентификаторов других ролей, которые включаются в данную роль          |
