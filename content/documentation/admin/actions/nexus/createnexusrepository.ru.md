---
title: CreateNexusRepository
weight: 10
---


{{< alert level="info" >}}
Для выполнения действия необходимо наличие токена — строки base64(`admin:password`), используемой как Basic Auth при запросах к Nexus.
{{< /alert >}}

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

| Поле          | Обязательность   | Описание                                                                                            | Пример                                 |
| ------------- | ---------------- | --------------------------------------------------------------------------------------------------- | -------------------------------------- |
| description   | Нет              | Документация по назначению этого действия/репозитория. Не используется самим Nexus, только для UI   | -                                      |
| name          | Да               | Название создаваемого репозитория. Должно быть уникальным в рамках Nexus                            | my-maven-repo                          |
| format        | Да               | Формат (`maven`, `docker`, `npm`, `raw` и т. д.)                                                    | maven                                  |
| type          | Да               | Тип: `hosted`, `proxy` или `group`                                                                  | hosted                                 |
| online        | Да               | Доступен ли репозиторий (`true`/`false`)                                                            | true                                   |
| storage       | Да               | Объект storage: `blobStoreName`, `strictContentTypeValidation`, `writePolicy`                       | [Пример](#пример-запроса-maven-hosted) |
| cleanup       | Нет              | Привязанные политики очистки (`policyNames`)                                                        | policyNames: [maven-cleanup]           |
| maven         | Для maven        | Только для maven: `versionPolicy`, `layoutPolicy`                                                   | [Пример](#пример-запроса-maven-hosted) |
| proxy         | Для proxy        | Прокси-репозиторий: `remoteUrl`, `contentMaxAge`, `metadataMaxAge`                                  | -                                      |
| group         | Для group        | Список значений `memberNames`                                                                       | [Пример](#пример-запроса-docker-group) |
| docker        | Для docker       | Специфичные для Docker параметры: `httpPort`, `v1Enabled`, `forceBasicAuth`                         | [Пример](#пример-запроса-docker-group) |
| component     | Очень редко      | Только для некоторых нестандартных сценариев                                                        | -                                      |
| attributes    | Нет              | Любые кастомные поля                                                                                | -                                      |

### Требования

- Используйте только те блоки (`maven`, `group`, `proxy`, `docker` и пр.), которые поддерживаются для вашего типа/формата.
- Для maven hosted обязательно `maven: {versionPolicy, layoutPolicy}`.
- Для group — обязательно `group.memberNames`.
- Для proxy — обязательно `proxy.remoteUrl`.
- Для docker — специфичные поля в `docker`.
