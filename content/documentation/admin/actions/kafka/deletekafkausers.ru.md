---
title: DeleteKafkaUsers
weight: 40
---


{{< alert level="info" >}}
Для выполнения действий необходимы учётные данные:
* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.
{{< /alert >}}

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

| Название            | Обязательность   | Описание                                                                                                                                                                                                                          | Возможные значения                        |
| ------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| securityProtocol    | Да               | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL       |
| saslMechanism       | Нет              | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512       |
| users               | Да               | Набор пользователей, которых необходимо удалить                                                                                                                                                                                   | -                                         |
| users.user          | Да               | Имя удаляемого пользователя                                                                                                                                                                                                       | -                                         |
| users.mechanism     | Да               | Механизм аутентификации удаляемого пользователя                                                                                                                                                                                   | SCRAM-SHA-256, SCRAM-SHA-512              |
