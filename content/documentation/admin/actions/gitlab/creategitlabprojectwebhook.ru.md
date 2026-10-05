---
title: CreateGitlabProjectWebhook
weight: 30
---

{{< alert level="info" >}}
Для выполнения действия требуется токен пользователя, от имени которого оно будет выполнено.
{{< /alert >}}

CreateGitlabProjectWebhook — создаёт вебхук в проекте GitLab.

### Пример запроса

```yaml
project_id: '0'
url: https://example.com
push_events: true
issues_events: true
merge_requests_events: true
pipeline_events: true
```

### Спецификация запроса

| Название                | Обязательность     | Описание                                                     |
| ----------------------- | ------------------ | ------------------------------------------------------------ |
| project_id              | Да                 | Идентификатор проекта, в котором необходимо создать вебхук   |
| url                     | Да                 | URL-адрес вебхука                                            |
| push_events             | Нет                | Запускать вебхук при push в репозиторий. По умолчанию `false` |
| issues_events           | Нет                | Запускать вебхук при создании Issue. По умолчанию `false` |
| merge_requests_events   | Нет                | Запускать вебхук при создании Merge Request. По умолчанию `false` |
| pipeline_events         | Нет                | Запускать вебхук при запуске Pipeline. По умолчанию `false` |

### Примечание

Действие осуществляет POST-запрос по URL: `/api/v4/projects/:id/hooks`.
