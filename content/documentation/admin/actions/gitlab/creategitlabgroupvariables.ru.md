---
title: CreateGitlabGroupVariables
weight: 80
---

{{< alert level="info" >}}
Для выполнения действия требуется токен пользователя, от имени которого оно будет выполнено.
{{< /alert >}}

CreateGitlabGroupVariables — создаёт переменные (variables) на уровне группы в GitLab.

### Пример запроса

```yaml
group_id: '0'
variables:
  - key: EXAMPLE_VARIABLE
    value: value
```

### Спецификация запроса

| Название        | Обязательность | Описание                                                                     |
|-----------------|----------------|------------------------------------------------------------------------------|
| group_id        | Да             | Идентификатор группы, в котором необходимо создать переменные                |
| variables       | Да             | Список создаваемых переменных                                                |

Список полей для переменных соответствует официальному GitLab Group-level Variables API, `/groups/:id/variables`, подробнее — [в документации GitLab](https://docs.gitlab.com/api/group_level_variables/#create-variable).

### Примечание

Действие осуществляет POST-запрос по URL: `/api/v4/groups/:id/variables`.
