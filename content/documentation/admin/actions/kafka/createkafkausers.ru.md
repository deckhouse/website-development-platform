---
title: CreateKafkaUsers
weight: 30
---


{{< alert level="info" >}}
Для выполнения действий необходимы учётные данные:
* `user` — имя пользователя, от которого будет запускаться выполнение действия.
* `password` — пароль пользователя, от имени которого будет запускаться выполнение действия.
{{< /alert >}}

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

| Название            | Обязательность   | Описание                                                                                                                                                                                                                          | Возможные значения                        |
| ------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| securityProtocol    | Да               | Протокол для подключения к Kafka. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL       |
| saslMechanism       | Нет              | Механизм аутентификации, который будет использовать SASL. Обязателен при использовании протокола SASL_PLAINTEXT или SASL_SSL. Подробнее — [в документации Kafka](https://kafka.apache.org/documentation/#security_sasl_mechanism) | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512       |
| users               | Да               | Набор пользователей, которых необходимо создать                                                                                                                                                                                   | -                                         |
| users.user          | Да               | Имя создаваемого пользователя                                                                                                                                                                                                     | -                                         |
| users.password      | Да               | Пароль создаваемого пользователя                                                                                                                                                                                                  | -                                         |
| users.mechanism     | Да               | Механизм аутентификации создаваемого пользователя                                                                                                                                                                                 | SCRAM-SHA-256, SCRAM-SHA-512              |
| users.iterations    | Да               | Количество итераций, которые будут применяться для хеширования пароля                                                                                                                                                             | От 4096 до 16384                          |
