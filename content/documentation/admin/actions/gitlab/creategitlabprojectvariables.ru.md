---
title: CreateGitlabProjectVariables
weight: 90
---

{{< alert level="info" >}}
Для выполнения действия требуется токен пользователя, от имени которого оно будет выполнено.
{{< /alert >}}

CreateGitlabProjectVariables — создаёт переменные на уровне проекта в GitLab.

### Пример запроса

```yaml
project_id: '0'
variables:
  - key: EXAMPLE_VARIABLE
    value: value
```

### Спецификация запроса

| Название          | Обязательность   | Описание                                                                       |
| ----------------- | ---------------- | ------------------------------------------------------------------------------ |
| project_id        | Да               | Идентификатор проекта, в котором необходимо создать переменные                 |
| variables         | Да               | Список создаваемых переменных                                                  |

Список полей для переменных соответствует официальному GitLab Project-level CI/CD variables API, `/projects/:id/variables`, подробнее — [в документации GitLab](https://docs.gitlab.com/api/project_level_variables/#create-a-variable).

### Примечание

Действие осуществляет POST-запрос по URL: `/api/v4/projects/:id/variables`.
