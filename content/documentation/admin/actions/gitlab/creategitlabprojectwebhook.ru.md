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
| push_events             | Да                 | Запускать вебхук при push в репозиторий                      |
| issues_events           | Да                 | Запускать вебхук при создании Issue                          |
| merge_requests_events   | Да                 | Запускать вебхук при создании Merge Request                  |
| pipeline_events         | Да                 | Запускать вебхук при запуске Pipeline                        |

### Примечание

Действие осуществляет POST-запрос по URL: `/api/v4/projects/:id/hooks`.
