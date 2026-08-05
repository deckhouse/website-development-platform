---
title: CreateNexusPrivilege
weight: 30
---


{{< alert level="info" >}}
Для выполнения действия необходимо наличие токена — строки base64(`admin:password`), используемой как Basic Auth при запросах к Nexus.
{{< /alert >}}

CreateNexusPrivilege — создаёт новую привилегию в Nexus Repository Manager 3. Привилегии определяют права доступа к репозиториям и другим ресурсам Nexus.

### Пример запроса (repository-view)

```yaml
name: example-privilege
description: Example privilege description
type: repository-view
actions:
  - READ
  - BROWSE
format: maven2
repository: maven-releases
```

### Пример запроса (repository-content-selector)

```yaml
name: content-selector-privilege
description: Privilege with content selector
type: repository-content-selector
actions:
  - READ
format: maven2
repository: maven-releases
contentSelector: my-content-selector
```

### Пример запроса (wildcard)

```yaml
name: wildcard-privilege
description: Wildcard privilege
type: wildcard
pattern: nx-*
actions:
  - READ
```

### Спецификация запроса

| Поле              | Обязательность     | Описание                                                                                                                                                            |
| ----------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name              | Да                 | Название создаваемой привилегии. Должно быть уникальным в рамках Nexus                                                                                              |
| description       | Нет                | Описание привилегии                                                                                                                                                 |
| type              | Да                 | Тип привилегии: `repository-view`, `repository-content-selector`, `repository-admin`, `application`, `wildcard`                                                     |
| actions           | Нет                | Список действий, разрешённых привилегией (например, `READ`, `BROWSE`, `CREATE`, `UPDATE`, `DELETE`)                                                                 |
| format            | Нет                | Формат репозитория (например, `maven2`, `docker`, `npm`). Используется для типов `repository-view`, `repository-content-selector`, `repository-admin`               |
| repository        | Нет                | Название репозитория. Используется для типов `repository-view`, `repository-content-selector`, `repository-admin`                                                   |
| contentSelector   | Нет                | Название селектора контента. Обязателен для типа `repository-content-selector`. Если не указан или невалиден, тип автоматически преобразуется в `repository-view`   |
| pattern           | Нет                | Шаблон для типа `wildcard`                                                                                                                                          |
| domain            | Нет                | Домен для типа `application`                                                                                                                                        |
| attributes        | Нет                | Дополнительные параметры в формате ключ-значение                                                                                                                    |

### Примечание

Для типа `repository-content-selector` селектор контента должен существовать в Nexus до создания привилегии. Если селектор контента не указан или невалиден, действие автоматически преобразует тип привилегии в `repository-view`.
