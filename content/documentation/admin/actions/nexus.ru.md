# Nexus Repository

## Содержание

- CreateNexusRepository

## CreateNexusRepository

`CreateNexusRepository` — создаёт новый репозиторий любого поддерживаемого типа (maven, docker, npm и др.) в Nexus Repository Manager 3 с помощью REST API.  
Параметры формата, типа и другие ключевые настройки полностью настраиваются и соответствуют Nexus API.

### Пример запроса (Maven hosted)

```yaml
description: |
  Maven hosted repo for internal Java build artifacts.
name: my-maven-repo
format: maven
type: hosted
online: true
storage:
  blobStoreName: default
  strictContentTypeValidation: true
  writePolicy: ALLOW
cleanup:
  policyNames:
    - maven-cleanup
maven:
  versionPolicy: RELEASE
  layoutPolicy: PERMISSIVE
```

### Пример запроса (Docker group)

```yaml
description: |
  Docker group repo aggregating hosted+proxy.
name: my-docker-group
format: docker
type: group
online: true
storage:
  blobStoreName: default
  strictContentTypeValidation: true
group:
  memberNames:
    - docker-hosted
    - docker-proxy
docker:
  v1Enabled: false
  forceBasicAuth: true
  httpPort: 5001
```

### Спецификация запроса

| Поле        | Обязательность  | Описание                                                                                          | Пример                                 |
|-------------|-----------------|---------------------------------------------------------------------------------------------------|----------------------------------------|
| `description` | Нет    | Документация по назначению этого действия/репозитория. Не используется самим Nexus, только для UI | -                                      |
| `name`      | Да | Название создаваемого репозитория. Должно быть уникальным в рамках Nexus                          | my-maven-repo                          |
| `format`    | Да | Формат (`maven`, `docker`, `npm`, `raw` и т. д.)                                                  | maven                                  |
| `type`      | Да | Тип: `hosted`, `proxy` или `group`                                                                | hosted                                 |
| `online`    | Да | Доступен ли репозиторий (`true`/`false`)                                                          | true                                   |
| `storage`   | Да | Объект storage: `blobStoreName`, `strictContentTypeValidation`, `writePolicy`                     | [Пример](#пример-запроса-maven-hosted) |
| `cleanup`   | Нет     | Привязанные политики очистки (`policyNames`)                                                      | policyNames: [maven-cleanup]           |
| `maven`     | Для maven       | Только для maven: `versionPolicy`, `layoutPolicy`                                                 | [Пример](#пример-запроса-maven-hosted) |
| `proxy`     | Для proxy       | Прокси-репозиторий: `remoteUrl`, `contentMaxAge`, `metadataMaxAge`                                | -                                      |
| `group`     | Для group       | Список включённых memberNames                                                                     | [Пример](#пример-запроса-docker-group) |
| `docker`    | Для docker      | docker-specific: `httpPort`, `v1Enabled`, `forceBasicAuth`                                        | [Пример](#пример-запроса-docker-group) |
| `component` | Очень редко     | Только для некоторых нестандартных сценариев                                                      | -                                      |
| `attributes`| Нет     | Любые кастомные поля                                                                              | -                                      |

### Требования

- Используйте только те блоки (`maven`, `group`, `proxy`, `docker` и пр.), которые поддерживаются для вашего типа/формата.
- Для maven hosted обязательно `maven: {versionPolicy, layoutPolicy}`.
- Для group — обязательно `group.memberNames`.
- Для proxy — обязательно `proxy.remoteUrl`.
- Для docker — специфичные поля в `docker`.

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

## DeleteNexusRepository

`DeleteNexusRepository` — удаляет существующий репозиторий из Nexus Repository Manager 3.

### Пример запроса

```yaml
name: my-repo-to-delete
```

### Спецификация запроса

| Поле | Обязательность  | Описание                                 |
|------|-----------------|------------------------------------------|
| `name` | Да | Название репозитория, который требуется удалить |

### Алгоритм

- Выполняется `DELETE` по адресу `/service/rest/v1/repositories/{name}`, где `{name}` — это значение поля `name`.
- Если репозиторий найден и удалён — возвращается 204.
- Если не найден — возвращается ошибка 404.

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

## CreateNexusPrivilege

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

| Поле             | Обязательность  | Описание                                                                                                                                                          |
|------------------|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name             | Да | Название создаваемой привилегии. Должно быть уникальным в рамках Nexus                                                                                            |
| description      | Нет     | Описание привилегии                                                                                                                                               |
| type             | Да | Тип привилегии: `repository-view`, `repository-content-selector`, `repository-admin`, `application`, `wildcard`                                                   |
| actions          | Нет     | Список действий, разрешённых привилегией (например, `READ`, `BROWSE`, `CREATE`, `UPDATE`, `DELETE`)                                                               |
| format           | Нет     | Формат репозитория (например, `maven2`, `docker`, `npm`). Используется для типов `repository-view`, `repository-content-selector`, `repository-admin`             |
| repository       | Нет     | Название репозитория. Используется для типов `repository-view`, `repository-content-selector`, `repository-admin`                                                 |
| contentSelector  | Нет     | Название селектора контента. Обязателен для типа `repository-content-selector`. Если не указан или невалиден, тип автоматически преобразуется в `repository-view` |
| pattern          | Нет     | Шаблон для типа `wildcard`                                                                                                                                        |
| domain           | Нет     | Домен для типа `application`                                                                                                                                      |
| attributes       | Нет     | Дополнительные атрибуты в формате ключ-значение                                                                                                                   |

### Примечание

Для типа `repository-content-selector` селектор контента должен существовать в Nexus до создания привилегии. Если селектор контента не указан или невалиден, действие автоматически преобразует тип привилегии в `repository-view`.

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

## AssignNexusPrivilege

AssignNexusPrivilege — назначает привилегии существующей роли в Nexus Repository Manager 3. Действие получает текущую конфигурацию роли и объединяет существующие привилегии с новыми.

### Пример запроса

```yaml
roleId: example-role
privileges:
  - example-privilege
  - another-privilege
```

### Спецификация запроса

| Поле       | Обязательность  | Описание                                                                     |
|------------|-----------------|------------------------------------------------------------------------------|
| roleId     | Да | Идентификатор роли, которой назначаются привилегии                            |
| privileges | Да | Список названий привилегий, которые необходимо назначить роли                |

### Алгоритм работы

1. Получает текущую конфигурацию роли из Nexus.
1. Объединяет существующие привилегии роли с новыми привилегиями из запроса.
1. Обновляет роль с объединённым списком привилегий.

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

### Примечание

Роль должна существовать в Nexus до назначения привилегий. Если роль не найдена, действие завершится с ошибкой. Все указанные привилегии также должны существовать в Nexus.

## DeleteNexusPrivilege

DeleteNexusPrivilege — удаляет привилегию из Nexus Repository Manager 3.

### Пример запроса

```yaml
name: example-privilege
```

### Спецификация запроса

| Поле | Обязательность  | Описание                                    |
|------|-----------------|---------------------------------------------|
| name | Да | Название привилегии, которую требуется удалить |

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

## CreateNexusRole

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

| Поле        | Обязательность  | Описание                                                                     |
|-------------|-----------------|------------------------------------------------------------------------------|
| id          | Да | Уникальный идентификатор роли                                                |
| name        | Да | Название роли                                                                |
| description | Нет     | Описание роли                                                                |
| privileges  | Нет     | Список названий привилегий, которые назначаются роли                         |
| roles       | Нет     | Список идентификаторов других ролей, которые включаются в данную роль        |

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

## AssignNexusRole

AssignNexusRole — назначает роли существующему пользователю в Nexus Repository Manager 3. Действие получает текущую конфигурацию пользователя и объединяет существующие роли с новыми.

### Пример запроса

```yaml
userId: example-user
roles:
  - example-role
  - another-role
```

### Спецификация запроса

| Поле   | Обязательность  | Описание                                                      |
|--------|-----------------|---------------------------------------------------------------|
| userId | Да | Идентификатор пользователя, которому назначаются роли          |
| roles  | Да | Список идентификаторов ролей, которые необходимо назначить пользователю |

### Алгоритм работы

1. Получает текущую конфигурацию пользователя из Nexus.
1. Объединяет существующие роли пользователя с новыми ролями из запроса.
1. Обновляет пользователя с объединённым списком ролей.

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

### Примечание

Пользователь должен существовать в Nexus до назначения ролей. Если пользователь не найден, действие завершится с ошибкой. Все указанные роли также должны существовать в Nexus.

## DeleteNexusRole

DeleteNexusRole — удаляет роль из Nexus Repository Manager 3.

### Пример запроса

```yaml
id: example-role
```

### Спецификация запроса

| Поле | Обязательность  | Описание                                |
|------|-----------------|-----------------------------------------|
| id   | Да | Идентификатор роли, которую требуется удалить |

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

## CreateNexusUser

CreateNexusUser — создаёт нового пользователя в Nexus Repository Manager 3.

### Пример запроса

```yaml
userId: example-user
firstName: First
lastName: Last
emailAddress: user@example.com
password: password
status: active
roles:
  - nx-admin
```

### Спецификация запроса

| Поле        | Обязательность  | Описание                                                                     |
|-------------|-----------------|------------------------------------------------------------------------------|
| userId      | Да | Уникальный идентификатор пользователя                                        |
| firstName   | Да | Имя пользователя                                                             |
| lastName    | Да | Фамилия пользователя                                                         |
| emailAddress| Да | Email-адрес пользователя                                                     |
| password    | Да | Пароль пользователя                                                          |
| status      | Да | Статус пользователя: `active` или `disabled`                                |
| roles       | Нет     | Список идентификаторов ролей, которые назначаются пользователю при создании  |

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.

## DeleteNexusUser

DeleteNexusUser — удаляет пользователя из Nexus Repository Manager 3.

### Пример запроса

```yaml
userId: example-user
```

### Спецификация запроса

| Поле   | Обязательность  | Описание                                    |
|--------|-----------------|---------------------------------------------|
| userId | Да | Идентификатор пользователя, которого требуется удалить |

### Учётные данные

* `token` — строка base64(`admin:password`), используется как Basic Auth при запросах к Nexus.
