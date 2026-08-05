---
title: DeleteKafkaTopics
weight: 20
---


{{< alert level="info" >}}
Running this action requires credentials:
* `user` — the username on whose behalf the action will be run.
* `password` — the password of the user on whose behalf the action will be run.
{{< /alert >}}

DeleteKafkaTopics — deletes existing topics in Kafka.

### Request example

```yaml
securityProtocol: SASL_PLAINTEXT
saslMechanism: PLAIN
topics:
  - example_1
  - example_2
```

### Request specification

| Name                 | Required | Description                                                                                                                                                                                                                          | Possible values                            |
| -------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| securityProtocol     | Yes      | Protocol used to connect to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                  | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL       |
| saslMechanism        | No       | Authentication mechanism to be used by SASL. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism)        | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512       |
| topics               | Yes      | List of topic names to delete                                                                                                                                                                                                          | -                                         |
