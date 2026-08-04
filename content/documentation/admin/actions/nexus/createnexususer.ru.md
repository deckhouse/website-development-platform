---
title: CreateNexusUser
weight: 90
---


{{< alert level="info" >}}
Для выполнения действия необходимо наличие токена — строки base64(`admin:password`), используемой как Basic Auth при запросах к Nexus.
{{< /alert >}}

CreateNexusUser — создаёт нового пользователя в Nexus Repository Manager 3.

### Пример запроса

```yaml
userId: example-user
firstName: First
lastName: Last
emailAddress: user@example.com
password: password
status: active
roles:
  - nx-admin
```

### Спецификация запроса

| Поле        | Обязательность  | Описание                                                                     |
|-------------|-----------------|------------------------------------------------------------------------------|
| userId      | Да | Уникальный идентификатор пользователя                                        |
| firstName   | Да | Имя пользователя                                                             |
| lastName    | Да | Фамилия пользователя                                                         |
| emailAddress| Да | Email-адрес пользователя                                                     |
| password    | Да | Пароль пользователя                                                          |
| status      | Да | Статус пользователя: `active` или `disabled`                                |
| roles       | Нет     | Список идентификаторов ролей, которые назначаются пользователю при создании  |
