---
title: DeleteCodeScoringProject
weight: 20
---

{{< alert level="info" >}}
Действие не использует механизм учётных данных DDP. Если для доступа к CodeScoring нужна аутентификация, добавьте необходимый HTTP-заголовок, например `Authorization`, в настройках [URL и HTTP-заголовков](../overview/#url-и-http-заголовки). DDP передаст этот заголовок в запрос к CodeScoring.
{{< /alert >}}

DeleteCodeScoringProject — удаляет проект в системе CodeScoring по его ID.

### Пример запроса

```yaml
id: 1
```

### Спецификация запроса

| Название   | Обязательность   | Описание                      |
| ---------- | ---------------- | ----------------------------- |
| id         | Да               | ID проекта в CodeScoring      |
