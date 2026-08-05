---
title: CreateKafkaUsers
weight: 30
---


{{< alert level="info" >}}
This action requires the following credentials:
* `user` — the username under which the action runs.
* `password` — the password for that user.
{{< /alert >}}

CreateKafkaUsers — creates new SASL/SCRAM users in Kafka.

### Request example

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

### Request specification

| Name                | Required | Description                                                                                                                                                                                                                        | Possible values                            |
| ------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| securityProtocol    | Yes      | Protocol for connecting to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                 | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL       |
| saslMechanism       | No       | SASL authentication mechanism. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism)                   | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512       |
| users               | Yes      | Set of users to create                                                                                                                                                                                                               | -                                         |
| users.user          | Yes      | Username to create                                                                                                                                                                                                                   | -                                         |
| users.password      | Yes      | Password for the user                                                                                                                                                                                                                | -                                         |
| users.mechanism     | Yes      | Authentication mechanism for the user                                                                                                                                                                                               | SCRAM-SHA-256, SCRAM-SHA-512              |
| users.iterations    | Yes      | Number of iterations used to hash the password                                                                                                                                                                                       | From 4096 to 16384                        |
