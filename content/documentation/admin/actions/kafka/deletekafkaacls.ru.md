---
title: DeleteKafkaACLs
weight: 60
---


{{< alert level="info" >}}
Для выполнения действий необходимы учётные данные:
* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.
{{< /alert >}}

DeleteKafkaACLs — удаляет набор ACL в Kafka.

### Пример запроса

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
acls:
  - topics:
      - example_1
    allow:
      - User:principal_2
      - Group:principal_3
    allow_hosts:
      - 127.0.0.1
    deny:
      - User:principal_4
      - Group:principal_5
    deny_hosts:
      - 127.0.0.1
    ops:
      - CREATE
      - READ
      - WRITE
      - DELETE
      - DESCRIBE
      - DESCRIBE_CONFIGS
      - ALTER
    pattern: LITERAL
  - any_topic: true
    any_group: true
    any_transactional_id: true
    any_allow: true
    any_allow_hosts: true
    any_deny: true
    any_deny_hosts: true
    ops:
      - ANY
    pattern: ANY
  - any_resource: true
    any_allow: true
    any_allow_hosts: true
    any_deny: true
    any_deny_hosts: true
    ops:
      - ANY
    pattern: ANY
```

### Спецификация запроса

| Название                  | Обязательность | Описание                                                                                                                                                                                                                          | Возможные значения                                      | Значение по умолчанию  |
|---------------------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|------------------------|
| securityProtocol          | Да             | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                     | -                      |
| saslMechanism             | Нет            | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                     | -                      |
| acls                      | Да             | Набор ACL, которые необходимо удалить                                                                                                                                                                                             | -                                                       | -                      |
| acls.ops                  | Да             | Список операций, для которых будет удалено правило                                                                                                                                                                                | [Список возможных операций](#список-возможных-операций) | -                      |
| acls.pattern              | Да             | Тип шаблона                                                                                                                                                                                                                       | [Список возможных шаблонов](#список-возможных-шаблонов) | -                      |
| acls.any_resource         | Нет            | Все ресурсы                                                                                                                                                                                                                       | -                                                       | false                  |
| acls.topics               | Нет            | Список из названий топиков, для которых будет применяться правило                                                                                                                                                                 | -                                                       | -                      |
| acls.any_topic            | Нет            | Все топики                                                                                                                                                                                                                        | -                                                       | false                  |
| acls.groups               | Нет            | Список из названий групп, для которых будет применяться правило                                                                                                                                                                   | -                                                       | -                      |
| acls.any_group            | Нет            | Все топики                                                                                                                                                                                                                        | -                                                       | false                  |
| acls.transactional_ids    | Нет            | Список из ID транзакций, для которых будет применяться правило                                                                                                                                                                    | -                                                       | -                      |
| acls.any_transactional_id | Нет            | Любая транзакция                                                                                                                                                                                                                  | -                                                       | false                  |
| acls.tokens               | Нет            | Список токенов, для которых будет применяться правило                                                                                                                                                                             | -                                                       | -                      |
| acls.any_token            | Нет            | Все токены                                                                                                                                                                                                                        | -                                                       | false                  |
| acls.allow                | Нет            | Список принципалов (user, group), для которых разрешить правило                                                                                                                                                                   | -                                                       | -                      |
| acls.any_allow            | Нет            | Любой принципал (user, group)                                                                                                                                                                                                     | -                                                       | false                  |
| acls.allow_hosts          | Нет            | Список хостов, для которых разрешить операцию                                                                                                                                                                                     | -                                                       | -                      |
| acls.any_allow_hosts      | Нет            | Любой хост, с которого разрешено проводить операцию                                                                                                                                                                               | -                                                       | false                  |
| acls.deny                 | Нет            | Список принципалов (user, group), для которых будет запрещено правило                                                                                                                                                             | -                                                       | -                      |
| acls.any_deny             | Нет            | Любой принципал (user, group)                                                                                                                                                                                                     | -                                                       | false                  |
| acls.deny_hosts           | Нет            | Список хостов, для которых будет запрещено операцию                                                                                                                                                                               | -                                                       | -                      |
| acls.any_deny_hosts       | Нет            | Любой хост, с которого запрещено проводить операцию                                                                                                                                                                               | -                                                       | false                  |

### Список возможных шаблонов

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

### Список возможных операций

Подробное описание — [в документации Kafka](https://kafka.apache.org/39/documentation/#operations_resources_and_protocols).

Topics:

* ALL.
* ALTER.
* ALTER_CONFIGS.
* CREATE.
* DELETE.
* DESCRIBE.
* DESCRIBE_CONFIGS.
* READ.
* WRITE.

Group:

* ALL.
* DELETE.
* DESCRIBE.
* READ.

TransactionalID:

* ALL.
* DESCRIBE.
* WRITE.

Token:

* DESCRIBE.
