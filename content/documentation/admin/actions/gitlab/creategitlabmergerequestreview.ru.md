---
title: CreateGitlabMergeRequestReview
weight: 75
---

{{< alert level="info" >}}
Для выполнения действия необходимы учётные данные `password` — токен GitLab пользователя, от имени которого будет выполнено действие.
{{< /alert >}}

CreateGitlabMergeRequestReview — создаёт merge request в GitLab из уже существующей ветки. В интерфейсе действие называется «Создать MR в Gitlab для существующей ветки».

В отличие от [CreateGitlabMergeRequest](creategitlabmergerequest/), действие не создаёт ветку и не записывает файлы, а перед созданием проверяет открытые merge request проекта. Действие используется процессом [обновления микросервиса из шаблона](../../templates/updates/).

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

| Название                      | Обязательность | Описание                                                                                              | Значение по умолчанию |
| ----------------------------- | -------------- | ----------------------------------------------------------------------------------------------------- | --------------------- |
| skipReview                    | Нет            | При значении `true` действие завершается успешно без создания merge request                            | `false`               |
| sourceBranch                  | Да             | Существующая ветка с изменениями                                                                       | -                     |
| targetBranch                  | Нет            | Ветка, в которую предлагаются изменения. Если не задана и действие выполняется для сущности микросервиса, используется значение роли маппинга `microservice_default_branch` | `main`                |
| dedupBranchPrefix             | Нет            | Префикс веток для поиска открытых merge request                                                        | Значение «Префикс ветки обновления» реестра шаблонов, иначе `ddp/template-upgrade-` |
| repositoryRef.gitlabProjectId | Да             | Идентификатор проекта GitLab. Если не задан и действие выполняется для сущности микросервиса, используется ID проекта микросервиса | -                     |
| mergeRequestSpec.title        | Нет            | Заголовок merge request                                                                                | `Upgrade template (<sourceBranch>)` |
| mergeRequestSpec.description  | Нет            | Описание merge request                                                                                 | -                     |

### Алгоритм работы

DDP:

1. Запрашивает открытые merge request проекта, ветки которых начинаются с префикса `dedupBranchPrefix`.
1. Если открыт merge request из ветки `sourceBranch`, завершает действие успешно и возвращает этот merge request.
1. Если открыт merge request из другой ветки с тем же префиксом, завершает действие ошибкой `review_already_open: open review for branch <branch> blocks <sourceBranch>`.
1. Иначе создаёт merge request из `sourceBranch` в `targetBranch`.

Проверяется одна страница из 100 открытых merge request проекта.

### Ответ

Ответ действия содержит поля:

| Поле           | Описание                                                        |
| -------------- | --------------------------------------------------------------- |
| `skipped`      | `true`, если действие завершилось без создания merge request по `skipReview` |
| `deduplicated` | `true`, если возвращён уже открытый merge request               |
| `provider`     | Провайдер системы контроля версий, `gitlab`                     |
| `reviewId`     | Номер merge request (`iid`) в проекте                           |
| `reviewUrl`    | Ссылка на merge request                                         |
| `sourceBranch` | Ветка-источник                                                  |
| `targetBranch` | Целевая ветка                                                   |

### Примечание

Действие осуществляет GET- и POST-запросы по URL: `/api/v4/projects/:id/merge_requests`. Ошибка создания merge request возвращается с кодом `review_create_failed`.
