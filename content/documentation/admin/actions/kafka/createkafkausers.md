---
title: CreateKafkaUsers
weight: 30
---


{{< alert level="info" >}}
Running this action requires credentials:
* `user` — the username on whose behalf the action will be run.
* `password` — the password of the user on whose behalf the action will be run.
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
| securityProtocol    | Yes      | Protocol used to connect to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL       |
| saslMechanism       | No       | Authentication mechanism to be used by SASL. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism)      | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512       |
| users               | Yes      | Set of users to create                                                                                                                                                                                                                | -                                         |
| users.user          | Yes      | Name of the user to create                                                                                                                                                                                                            | -                                         |
| users.password      | Yes      | Password of the user to create                                                                                                                                                                                                        | -                                         |
| users.mechanism     | Yes      | Authentication mechanism of the user to create                                                                                                                                                                                        | SCRAM-SHA-256, SCRAM-SHA-512              |
| users.iterations    | Yes      | Number of iterations used for password hashing                                                                                                                                                                                        | From 4096 to 16384                        |
