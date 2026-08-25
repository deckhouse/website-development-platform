---
title: CreateGitlabMergeRequestReview
weight: 45
---

{{< alert level="info" >}}
Для выполнения действия требуется токен пользователя, от имени которого оно будет выполнено.
{{< /alert >}}

CreateGitlabMergeRequestReview — создаёт merge request в GitLab для уже существующей ветки.

В отличие от [CreateGitlabMergeRequest](../creategitlabmergerequest/), действие не создаёт ветку и проверяет, нет ли уже открытого merge request для этой работы. Если открытый merge request найден, новый не создаётся, а действие возвращает ссылку на существующий.

### Пример запроса

```yaml
skipReview: false
sourceBranch: ddp/template-upgrade-1.2.0
targetBranch: main
dedupBranchPrefix: ddp/template-upgrade-
repositoryRef:
  gitlabProjectId: "0"
mergeRequestSpec:
  title: Upgrade template
  description: Apply template upgrade changes
```

### Спецификация запроса

| Название                        | Обязательность | Описание                                                                                            |
| ------------------------------- | -------------- | --------------------------------------------------------------------------------------------------- |
| skipReview                      | Нет            | Если `true`, действие завершается успешно, не создавая merge request                                 |
| sourceBranch                    | Да             | Существующая ветка с изменениями                                                                     |
| targetBranch                    | Да             | Ветка, в которую предлагаются изменения                                                              |
| dedupBranchPrefix               | Нет            | Префикс ветки для поиска дубликатов: открытый merge request с веткой по этому префиксу блокирует создание нового |
| repositoryRef.gitlabProjectId   | Да             | Идентификатор проекта GitLab                                                                         |
| mergeRequestSpec.title          | Да             | Заголовок merge request                                                                              |
| mergeRequestSpec.description    | Нет            | Описание merge request                                                                               |

### Примечание

Проверка дубликатов работает так:

- если открытый merge request существует ровно для `sourceBranch` — действие возвращает его и завершается успешно;
- если задан `dedupBranchPrefix` и открытый merge request существует для другой ветки с этим префиксом — действие завершается ошибкой, чтобы не плодить параллельные обновления одного и того же вида.

Действие используется процессами [обновления сервиса из шаблона](../../../templates/#обновление-сервиса).
