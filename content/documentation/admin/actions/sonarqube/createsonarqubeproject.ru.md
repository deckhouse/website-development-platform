---
title: CreateSonarqubeProject
weight: 10
---

{{< alert level="info" >}}
Для выполнения действий необходимо наличие токена SonarQube с типом User Token, сгенерированного пользователем, от имени которого будет запускаться выполнение действия.
{{< /alert >}}

CreateSonarqubeProject — создаёт новый проект в SonarQube.
Действие использует SonarQube Web API для создания проекта с указанными параметрами, такими как ключ (project key), название проекта, главная ветка, параметры определения нового кода (new code definition) и видимость проекта. Аутентификация осуществляется с использованием токена SonarQube, который должен быть передан в учётных данных.

### Пример запроса

```yaml
project: example-project
name: example-project
mainBranch: develop
newCodeDefinitionType: NUMBER_OF_DAYS
newCodeDefinitionValue: '30'
visibility: public
```

### Спецификация запроса

| Название                | Обязательность | Описание                                                                                      | Возможные значения                                 | Значение по умолчанию   |
|-------------------------|----------------|-----------------------------------------------------------------------------------------------|----------------------------------------------------|-------------------------|
| project                 | Да             | Уникальный идентификатор проекта (Project Key) в SonarQube                                    | -                                                  | -                       |
| name                    | Да             | Название проекта, отображаемое в интерфейсе SonarQube                                         | -                                                  | -                       |
| mainBranch              | Нет            | Название главной ветки проекта                                                                | -                                                  | master                  |
| newCodeDefinitionType   | Нет            | Метод определения «нового кода»                                                               | PREVIOUS_VERSION, NUMBER_OF_DAYS, REFERENCE_BRANCH | -                       |
| newCodeDefinitionValue  | Нет            | Значение для определения «нового кода» (например, количество дней, если тип - NUMBER_OF_DAYS) | -                                                  | -                       |
| visibility              | Нет            | Видимость проекта                                                                             | private, public                                    | private                 |
