---
title: CreateKafkaACLs
weight: 50
---

{{< alert level="info" >}}
Для выполнения действия необходимы учётные данные: `user` (имя пользователя, от имени которого выполняется действие)
и `password` (пароль этого пользователя).
{{< /alert >}}

CreateKafkaACLs — создаёт набор ACL в Kafka.

## Пример запроса

Пример тела запроса действия:

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

## Спецификация запроса

Тело запроса действия поддерживает следующие параметры:

| Название                 | Обязательность   | Описание                                                                                                                                                                                                                            | Возможные значения                                        | Значение по умолчанию    |
| ------------------------ | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------ |
| `securityProtocol`       | Да               | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                  | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                       | —                        |
| `saslMechanism`          | Нет              | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism)   | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                       | —                        |
| `acls`                   | Да               | Набор ACL, которые необходимо создать                                                                                                                                                                                               | —                                                         | —                        |
| `acls.ops`               | Да               | Список операций, для которых будет создано правило                                                                                                                                                                                  | [Список возможных операций](#список-возможных-операций)   | —                        |
| `acls.pattern`           | Да               | Тип шаблона                                                                                                                                                                                                                         | [Список возможных шаблонов](#список-возможных-шаблонов)   | —                        |
| `acls.topics`            | Нет              | Список из названий топиков, для которых применяется правило                                                                                                                                                                         | —                                                         | —                        |
| `acls.groups`            | Нет              | Список из названий групп, для которых применяется правило                                                                                                                                                                           | —                                                         | —                        |
| `acls.transactional_ids` | Нет              | Список из ID транзакций, для которых применяется правило                                                                                                                                                                            | —                                                         | —                        |
| `acls.tokens`            | Нет              | Список токенов, для которых применяется правило                                                                                                                                                                                     | —                                                         | —                        |
| `acls.allow`             | Нет              | Список принципалов (user, group), для которых разрешается правило                                                                                                                                                                   | —                                                         | —                        |
| `acls.deny`              | Нет              | Список принципалов (user, group), для которых запрещается правило                                                                                                                                                                   | —                                                         | —                        |
| `acls.allow_hosts`       | Нет              | Список хостов, для которых разрешается операция                                                                                                                                                                                     | —                                                         | —                        |
| `acls.deny_hosts`        | Нет              | Список хостов, для которых запрещается операция                                                                                                                                                                                     | —                                                         | —                        |

## Список возможных шаблонов

Поддерживаются следующие типы шаблонов:

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

## Список возможных операций

Подробнее — [в документации Kafka](https://kafka.apache.org).

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

Tokens: DESCRIBE.
