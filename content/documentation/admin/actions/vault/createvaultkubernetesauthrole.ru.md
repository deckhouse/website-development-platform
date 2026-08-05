---
title: CreateVaultKubernetesAuthRole
weight: 30
---

{{< alert level="info" >}}
Для выполнения действия необходимо наличие Vault-токена, обладающего правами на создание/обновление ролей Kubernetes auth backend.
{{< /alert >}}

CreateVaultKubernetesAuthRole — создаёт или обновляет роль аутентификации Kubernetes в HashiCorp Vault.

### Пример запроса

```yaml
mountPath: kubernetes
role: example
bound_service_account_names:
  - default
bound_service_account_namespaces:
  - default
optional:
  token_ttl: 1h
  token_max_ttl: 12h
  audience: vault
  token_policies:
    - default
```

### Спецификация запроса

| Название                              | Обязательность   | Описание                                                                            |
| ------------------------------------- | ---------------- | ----------------------------------------------------------------------------------- |
| mountPath                             | Да               | Путь монтирования Kubernetes auth backend в Vault (например, kubernetes)            |
| role                                  | Да               | Название роли, которая создаётся в Vault                                            |
| bound_service_account_names           | Да               | Список имён service account'ов, которым разрешён доступ через данную роль           |
| bound_service_account_namespaces      | Да               | Список неймспейсов (namespaces), в которых разрешён доступ через данную роль        |
| optional                              | Нет              | Дополнительные параметры роли (приведены в следующей таблице)                       |

Поддерживаемые значения в optional:

| Поле                      | Тип             | Описание                                                                |
| ------------------------- | --------------- | ----------------------------------------------------------------------- |
| token_ttl                 | string          | Время жизни (TTL) токена, выданного при логине                          |
| token_max_ttl             | string          | Максимальное TTL токена                                                 |
| token_policies            | []string        | Дополнительные политики, назначаемые при логине                         |
| audience                  | string          | Значение JWT audience (aud), которое Vault ожидает от токена            |
| token_period              | string          | Периодичность выдачи токена                                             |
| token_explicit_max_ttl    | string          | Явный верхний предел TTL токена                                         |
| token_num_uses            | int             | Ограничение на количество использований токена                          |
| token_type                | string          | Тип выдаваемого токена (например, service, batch)                       |
| alias_name_source         | string          | Источник alias name для identity                                        |
| token_no_default_policy   | bool            | Исключение default policy из состава токена                             |
| token_bound_cidrs         | []string        | Ограничение CIDR-диапазонов, откуда можно использовать выданный токен   |

Полный список поддерживаемых параметров приведён в официальной документации [HashiCorp Vault](https://developer.hashicorp.com/vault/docs/auth/kubernetes#parameters).
