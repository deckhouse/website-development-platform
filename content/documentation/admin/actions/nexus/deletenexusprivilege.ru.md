---
title: DeleteNexusPrivilege
weight: 50
---


{{< alert level="info" >}}
Для выполнения действия необходимо наличие токена — строки base64(`admin:password`), используемой как Basic Auth при запросах к Nexus.
{{< /alert >}}

DeleteNexusPrivilege — удаляет привилегию из Nexus Repository Manager 3.

### Пример запроса

```yaml
name: example-privilege
```

### Спецификация запроса

| Поле   | Обязательность    | Описание                                       |
| ------ | ----------------- | ---------------------------------------------- |
| name   | Да                | Название привилегии, которую требуется удалить |
