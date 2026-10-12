---
title: API рабочих пространств
description: "Пользовательские API DDP для шаблонов, ревизий файлов, публикации, подключения, создания и управления жизненным циклом рабочих пространств."
weight: 110
---

Маршруты относятся к API Deckhouse Development Portal (DDP) `/api/v2` и требуют аутентификации и разрешений соответствующего действия. Работа с конкретным пространством дополнительно ограничена его владельцем.

## Шаблоны и файлы

Шаблон, черновик и опубликованная версия имеют разные UUID:

| Метод и путь относительно API v2 | Назначение |
| --- | --- |
| `GET /workspace-templates` | Доступные шаблоны |
| `GET /workspace-templates/admin` | Административный список |
| `GET /workspace-templates/:template_uuid` | Метаданные шаблона |
| `POST /workspace-templates` | Создание шаблона с явным сервисом и неймспейсом |
| `PATCH /workspace-templates/:template_uuid` | Метаданные и архив |
| `GET /workspace-templates/:template_uuid/drafts` | Черновики |
| `POST /workspace-templates/:template_uuid/drafts` | Новый черновик |
| `GET /workspace-templates/:template_uuid/drafts/:draft_uuid` | Ревизия, manifest и файлы |
| `PUT /workspace-templates/:template_uuid/drafts/:draft_uuid/files` | Сохранение набора файлов |
| `POST /workspace-templates/:template_uuid/drafts/:draft_uuid/inspect` | Статическая проверка ревизии |
| `POST /workspace-templates/:template_uuid/drafts/:draft_uuid/publish` | Подготовка и публикация |
| `GET /workspace-templates/:template_uuid/preparations` | История подготовки |
| `GET /workspace-templates/:template_uuid/preparations/:preparation_uuid` | Результат подготовки |
| `GET /workspace-templates/:template_uuid/versions` | Опубликованные версии |
| `GET /workspace-templates/:template_uuid/versions/:version_uuid` | Версия и декларации |
| `GET /workspace-templates/:template_uuid/versions/:version_uuid/files` | Исходники версии для администратора |
| `POST /workspace-templates/:template_uuid/activate` | Активация опубликованной версии |
| `GET /workspace-templates/:template_uuid/drafts/:draft_uuid/test-workspaces` | Последняя проверка черновика |
| `POST /workspace-templates/:template_uuid/drafts/:draft_uuid/test-workspaces` | Создание тестового пространства |
| `GET /workspace-templates/:template_uuid/drafts/:draft_uuid/test-workspaces/:check_uuid` | Результат конкретной проверки |

В JSON файла используются `path`, `content` и `executable`. `content` кодируется Base64 для сохранения произвольных байтов. Размер и digest manifest относятся к декодированному содержимому.

Сохранение передаёт `expectedRevision` и список `mutations`. Каждая операция содержит `operation`, `path`, при записи — `content` и `executable`; переименование дополнительно содержит `destination`. Конкурирующая правка даёт конфликт; загрузите актуальную ревизию перед повтором.

Публикация содержит:

```json
{
  "expectedRevision": 1,
  "label": "v1",
  "requestUuid": "<REQUEST_UUID>"
}
```

Замените `<REQUEST_UUID>` новым UUID запроса. Подготовка асинхронная; отслеживайте результат по preparation. Повтор того же request UUID с другими входами отклоняется.

## Настройки и квоты

Общие настройки доступны отдельными маршрутами:

| Метод и путь | Назначение |
| --- | --- |
| `GET /platform_settings/workspace_connection_settings` | Служебное подключение или `null` до настройки |
| `PUT /platform_settings/workspace_connection_settings` | Сохранение подключения |
| `GET /workspace-platform-policy` | Политика лимитов |
| `PUT /workspace-platform-policy` | Сохранение политики |
| `GET /ai/workspaces/limits` | Квоты и занятость пользователя |

Тело настройки подключения:

```json
{
  "externalServiceUuid": "<KUBERNETES_SERVICE_UUID>",
  "namespace": "ddp-jobs"
}
```

Замените UUID идентификатором внешнего сервиса DDP, а неймспейс — существующим служебным неймспейсом. Настройка содержит ссылки; пароль или токен кластера задаются через внешний сервис.

## Создание и состояние

Создание закрепляет конкретную активную версию:

```json
{
  "templateUuid": "<TEMPLATE_UUID>",
  "expectedActiveVersionUuid": "<VERSION_UUID>",
  "parameters": {},
  "kubernetesExternalServiceUuid": "<KUBERNETES_SERVICE_UUID>",
  "namespace": "team-workspaces",
  "gitlabServiceUuid": "<GITLAB_SERVICE_UUID>",
  "repositoryUrl": "https://gitlab.example.com/team/project.git",
  "baseRef": "main",
  "provider": "<AI_PROVIDER_IDENTIFIER>",
  "ttlHours": 24,
  "requestUuid": "<REQUEST_UUID>"
}
```

UUID и идентификатор провайдера относятся к доступным объектам DDP. Параметры соответствуют декларациям версии. Сервис и namespace должны совпадать с принятой привязкой шаблона; для `per_workspace` namespace задаёт DDP. Смена активной версии между загрузкой формы и созданием возвращает конфликт.

Для чтения состояния используются:

| Метод и путь | Назначение |
| --- | --- |
| `POST /ai/workspaces` | Создание |
| `GET /ai/workspaces` | Личные пространства |
| `GET /ai/workspaces/:workspace_uuid` | Подробности |
| `GET /ai/workspaces/:workspace_uuid/lifecycle` | Входы, операция, план и доступные действия |
| `GET /ai/workspaces/:workspace_uuid/builds` | История инфраструктурных операций |
| `GET /ai/workspaces/:workspace_uuid/builds/:build_uuid/events` | Ограниченный журнал операции |
| `GET /ai/workspaces/:workspace_uuid/turns` | Задания |
| `GET /ai/workspaces/:workspace_uuid/turns/:turn_uuid` | Результат задания |
| `GET /ai/workspaces/:workspace_uuid/turns/:turn_uuid/events` | События задания |

Возвращённые `capabilities` учитывают состояние и разрешения. Проверяйте их перед показом действий и загружайте новую lifecycle revision перед изменениями.

## Операции

Инфраструктурные действия выполняются через следующие маршруты:

| Метод и путь | Назначение |
| --- | --- |
| `POST /ai/workspaces/:workspace_uuid/stop` | Остановка |
| `POST /ai/workspaces/:workspace_uuid/restore` | Новый запуск сохранённой среды |
| `POST /ai/workspaces/:workspace_uuid/retry` | Повтор допустимой неудачной операции |
| `POST /ai/workspaces/:workspace_uuid/plans` | План версии и параметров |
| `POST /ai/workspaces/:workspace_uuid/plans/:build_uuid/apply` | Применение принятого плана |
| `POST /ai/workspaces/:workspace_uuid/delete` | Подтверждённое удаление |
| `POST /ai/workspaces/:workspace_uuid/reconnect` | Проверка соединения с той же средой |
| `POST /ai/workspaces/:workspace_uuid/ide` | Получение доступа к IDE |
| `POST /ai/workspaces/:workspace_uuid/message` | Отправка задания |
| `POST /ai/workspaces/:workspace_uuid/status` | Состояние сессии |

Restore, Retry и Delete используют `requestUuid`, `expectedRevision` и `confirmDelete`. Для Delete значение `confirmDelete = true`; обычные неразрушительные операции используют `false`. Форма Stop и подключение IDE имеют собственные тела запроса; используйте схему соответствующего метода.

Запрос плана дополнительно содержит `versionUuid` и `parameters`. Применение использует UUID plan build, актуальную revision и принятые решения:

| Поле Apply | Назначение |
| --- | --- |
| `requestUuid` | Идентификатор запроса |
| `expectedRevision` | Актуальная revision lifecycle |
| `confirmDelete` | `false` для обновления |
| `createScriptDigests` | Согласованные имена и SHA-256 скриптов подготовки |
| `fileResolutions` | Выбор `preserve/overwrite` и хеши конкретного проверенного файла |

Решение файла содержит `choice`, `currentExists`, `currentDigest` и `templateDigest`. Передавайте значения из актуального плана. Изменившийся файл или state требует нового планирования.

Успешное принятие асинхронной операции не означает завершение инфраструктуры. Проверяйте состояние операции и истории; повтор HTTP-запроса не должен создавать новое задание или автоматически повторять поручение модели.

## Инструменты чата

При прикреплённом пространстве ассистент использует инструменты MCP:

| Инструмент | Вход | Результат |
| --- | --- | --- |
| `workspace_run_task` | `task`, до `8192` байт UTF-8 | Задание в пространстве текущего чата и его результат |
| `workspace_get_task` | `turnUuid` | Состояние и результат ранее принятого задания |

Идентификаторы запросов и привязку к пространству задаёт DDP. Модель передаёт задачу, затем проверяет полученный turn UUID при `queued/running`. Повторная отправка той же работы вместо чтения результата не требуется.
