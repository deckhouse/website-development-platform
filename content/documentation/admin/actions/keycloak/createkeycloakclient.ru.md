---
title: CreateKeycloakClient
weight: 10
---


{{< alert level="info" >}}
Для выполнения действия необходимы учётные данные:
* `username` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.
{{< /alert >}}

CreateKeycloakClient — создаёт нового клиента в Keycloak.

### Пример запроса

```yaml
realm: master
config:
  clientId: example
  name: example
  enabled: true
  clientAuthenticatorType: client-secret
  secret: secret
  defaultClientScopes:
  - roles
  - profile
  - email
  optionalClientScopes:
  - address
  - phone
  - offline_access
```

### Спецификация запроса

| Название | Обязательность | Описание                                                                                     | Возможные значения                                                                                |
|----------|----------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| realm    | Да             | Realm в Keycloak, где требуется создать клиента                                              | -                                                                                                 |
| config   | Да             | Параметры создаваемого клиента в соответствии со [спецификацией ClientRepresentation Keycloak](https://www.keycloak.org/docs-api/latest/rest-api/index.html#ClientRepresentation) | -                                                                                                 |
