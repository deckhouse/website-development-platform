---
title: CreateKeycloakClient
weight: 10
---

{{< alert level="info" >}}
Для выполнения действия необходимы учётные данные: `username` (имя пользователя, от имени которого выполняется действие)
и `password` (пароль этого пользователя).

Портал получает токен через клиент `admin-cli` в realm, указанном в `realm`, — в том же realm, где создаётся клиент.
Пользователь должен существовать в этом realm и иметь права на создание в нём клиентов.
{{< /alert >}}

CreateKeycloakClient — создаёт нового клиента в Keycloak.

## Пример запроса

Пример тела запроса действия:

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

## Спецификация запроса

Тело запроса действия поддерживает следующие параметры:

| Название   | Обязательность   | Описание                                                                                                                                                                          | Возможные значения                                                                                  |
| ---------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `realm`    | Да               | Realm в Keycloak, где требуется создать клиента                                                                                                                                   | —                                                                                                   |
| `config`   | Да               | Параметры создаваемого клиента в соответствии со [спецификацией ClientRepresentation Keycloak](https://www.keycloak.org/docs-api/latest/rest-api/index.html#ClientRepresentation) | —                                                                                                   |
