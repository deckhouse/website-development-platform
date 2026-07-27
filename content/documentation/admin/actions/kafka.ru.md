---
title: Kafka
---

# Kafka

## Топики

- CreateKafkaTopics
- DeleteKafkaTopics

## Пользователи

- CreateKafkaUsers
- DeleteKafkaUsers

## ACL

- CreateKafkaACLs
- DeleteKafkaACLs

## CreateKafkaTopics

CreateKafkaTopics — создаёт новые топики в Kafka.

### Пример запроса

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
partitions: 1
replication_factor: 1
configs: {}
topics:
  - example_1
  - example_2
```

### Спецификация запроса

| Название                | Обязательность | Описание                                                                                                                                                                                                   | Возможные значения                                                                              |
|-------------------------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| securityProtocol       | Да             | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                         | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                                                             |
| saslMechanism          | Нет            | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                                                             |
| partitions              | Да             | Количество разделов (партиций), на которые будет разделён топик                                                                                                                                            | -                                                                                               |
| replication_factor      | Да             | Количество копий (реплик) каждой партиции топика, которые необходимо разместить на разных брокерах                                                                                                         | -                                                                                               |
| configs                 | Да             | Конфигурация в формате ключ-значение для создаваемых топиков                                                                                                                                               | Значения приведены [в документации Kafka](https://kafka.apache.org/documentation/#topicconfigs) |
| topics                  | Да              | Список названий топиков, которые необходимо создать                                                                                                                                                        | -                                                                                               |

### Учётные данные

* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.

## DeleteKafkaTopics

DeleteKafkaTopics — удаляет существующие топики в Kafka.

### Пример запроса

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
topics:
  - example_1
  - example_2
```

### Спецификация запроса

| Название                  | Обязательность | Описание                                            | Возможные значения                      |
|---------------------------|----------------|-----------------------------------------------------|-----------------------------------------|
| auth_type                 | Да             | Тип авторизации в Kafka                             | PLAINTEXT, SCRAM-SHA-256, SCRAM-SHA-512 |
| topics                    | Да             | Список названий топиков, которые необходимо удалить | -                                       |

### Учётные данные

* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.

## CreateKafkaUsers

CreateKafkaUsers — создаёт новых пользователей SASL/SCRAM в Kafka.

### Пример запроса

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
users:
  - user: example_user_1
    password: example_password_user_1
    mechanism: SCRAM-SHA-256
    iterations: 4096
  - user: example_user_2
    password: example_password_user_2
    mechanism: SCRAM-SHA-256
    iterations: 4096
```

### Спецификация запроса

| Название          | Обязательность | Описание                                                                                                                                                                                                  | Возможные значения                      |
|-------------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| securityProtocol | Да             | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                        | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL     |
| saslMechanism    | Нет            | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512     |
| users             | Да             | Набор пользователей, которых необходимо создать                                                                                                                                                           | -                                       |
| users.user        | Да             | Имя создаваемого пользователя                                                                                                                                                                             | -                                       |
| users.password    | Да             | Пароль создаваемого пользователя                                                                                                                                                                          | -                                       |
| users.mechanism   | Да             | Механизм аутентификации создаваемого пользователя                                                                                                                                                         | SCRAM-SHA-256, SCRAM-SHA-512            |
| users.iterations  | Да              | Количество итераций, которые будут применяться для хеширования пароля                                                                                                                                     | От 4096 до 16384                        |

### Учётные данные

* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.

## DeleteKafkaUsers

DeleteKafkaUsers — удаляет существующих пользователей SASL/SCRAM в Kafka.

### Пример запроса

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
users:
  - user: example_user_1
    mechanism: SCRAM-SHA-256
  - user: example_user_2
    mechanism: SCRAM-SHA-256
```

### Спецификация запроса

| Название          | Обязательность | Описание                                                                                                                                                                                                   | Возможные значения                      |
|-------------------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| securityProtocol | Да             | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL     |
| saslMechanism    | Нет            | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512     |
| users             | Да             | Набор пользователей, которых необходимо удалить                                                                                                                                                            | -                                       |
| users.user        | Да             | Имя удаляемого пользователя                                                                                                                                                                                | -                                       |
| users.mechanism   | Да              | Механизм аутентификации удаляемого пользователя                                                                                                                                                            | SCRAM-SHA-256, SCRAM-SHA-512            |

### Учётные данные

* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.

## CreateKafkaACLs

CreateKafkaACLs — создаёт новый набор ACL в Kafka.

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
    deny:
      - User:principal_4
      - Group:principal_5
    ops:
      - CREATE
      - READ
      - WRITE
      - DELETE
      - DESCRIBE
      - DESCRIBE_CONFIGS
      - ALTER
    pattern: LITERAL
  - topics:
      - example_6
    allow:
      - User:principal_7
    allow_hosts:
      - 127.0.0.1
    deny:
      - User:principal_8
    deny_hosts:
      - 127.0.0.1
    ops:
      - CREATE
    pattern: LITERAL
```

### Спецификация запроса

| Название                | Обязательность | Описание                                                                                                                                                                                                                          | Возможные значения                                      | Значение по умолчанию  |
|-------------------------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|------------------------|
| securityProtocol        | Да             | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                     | -                      |
| saslMechanism           | Нет            | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                     | -                      |
| acls                    | Да             | Набор ACL, которые необходимо создать                                                                                                                                                                                             | -                                                       | -                      |
| acls.ops                | Да             | Список операций, для которых будет создано правило                                                                                                                                                                                | [Список возможных операций](#список-возможных-операций) | -                      |
| acls.pattern            | Да             | Тип шаблона                                                                                                                                                                                                                       | [Список возможных шаблонов](#список-возможных-шаблонов) | -                      |
| acls.topics             | Нет            | Список из названий топиков, для которых применяется правило                                                                                                                                                                       | -                                                       | -                      |
| acls.groups             | Нет            | Список из названий групп, для которых применяется правило                                                                                                                                                                         | -                                                       | -                      |
| acls.transactional_ids  | Нет            | Список из ID транзакций, для которых применяется правило                                                                                                                                                                          | -                                                       | -                      |
| acls.tokens             | Нет            | Список токенов, для которых применяется правило                                                                                                                                                                                   | -                                                       | -                      |
| acls.allow              | Нет            | Список хостов, для которых разрешается операция                                                                                                                                                                                   | -                                                       | -                      |
| acls.deny               | Нет            | Список принципалов (user, group), для которых запрещается правило                                                                                                                                                                 | -                                                       | -                      |
| acls.deny_hosts         | Нет            | Список хостов, для которых запрещается операция                                                                                                                                                                                   | -                                                       | -                      |

### Учётные данные

* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.

### Список возможных шаблонов

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

### Список возможных операций

Подробнее — [в документации Kafka](https://kafka.apache.org).

Topic:

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

Tokens:

* DESCRIBE.

## DeleteKafkaACLs

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
    deny_host:
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

| Название                  | Обязательность | Описание                                                                                                                                                                                                   | Возможные значения                                      | Значение по умолчанию  |
|---------------------------|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|------------------------|
| securityProtocol          | Да             | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                     | -                      |
| saslMechanism             | Нет            | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                     | -                      |
| acls                      | Да             | Набор ACL, которые необходимо создать                                                                                                                                                                      | -                                                       | -                      |
| acls.ops                  | Да             | Список операций, для которых будет создано правило                                                                                                                                                         | [Список возможных операций](#список-возможных-операций) | -                      |
| acls.pattern              | Да             | Тип шаблона                                                                                                                                                                                                | [Список возможных шаблонов](#список-возможных-шаблонов) | -                      |
| acls.any_resource         | Нет            | Все ресурсы                                                                                                                                                                                                | -                                                       | false                  |
| acls.topics               | Нет            | Список из названий топиков, для которых применять правило                                                                                                                                                  | -                                                       | -                      |
| acls.any_topic            | Нет            | Все топики                                                                                                                                                                                                 | -                                                       | false                  |
| acls.groups               | Нет            | Список из названий групп, для которых применять правило                                                                                                                                                    | -                                                       | -                      |
| acls.any_group            | Нет            | Все топики                                                                                                                                                                                                 | -                                                       | false                  |
| acls.transactional_ids    | Нет            | Список из ID транзакций, для которых применять правило                                                                                                                                                     | -                                                       | -                      |
| acls.any_transactional_id | Нет            | Любая транзакция                                                                                                                                                                                           | -                                                       | false                  |
| acls.tokens               | Нет            | Список токенов, для которых применять правило                                                                                                                                                              | -                                                       | -                      |
| acls.any_token            | Нет            | Все токены                                                                                                                                                                                                 | -                                                       | false                  |
| acls.allow                | Нет            | Список принципалов (user, group), для которых разрешить правило                                                                                                                                            | -                                                       | -                      |
| acls.any_allow            | Нет            | Любой принципал (user, group)                                                                                                                                                                              | -                                                       | false                  |
| acls.allow_hosts          | Нет            | Список хостов, для которых разрешить операцию                                                                                                                                                              | -                                                       | -                      |
| acls.any_allow_hosts      | Нет            | Любой хост, с которого разрешено проводить операцию                                                                                                                                                        | -                                                       | false                  |
| acls.deny                 | Нет            | Список принципалов (user, group), для которых запретить правило                                                                                                                                            | -                                                       | -                      |
| acls.any_deny             | Нет            | Любой принципал (user, group)                                                                                                                                                                              | -                                                       | false                  |
| acls.deny_hosts           | Нет            | Список хостов, для которых запретить операцию                                                                                                                                                              | -                                                       | -                      |
| acls.any_deny_hosts       | Нет            | Любой хост, с которого запрещено проводить операцию                                                                                                                                                        | -                                                       | false                  |

### Учётные данные

* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.

### Список возможных pattern

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

### Список возможных операций

Подробное описание — [в документации Kafka](https://kafka.apache.org/39/documentation/#operations_resources_and_protocols).

Topic:

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
