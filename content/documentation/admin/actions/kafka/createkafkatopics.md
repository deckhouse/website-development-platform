---
title: CreateKafkaTopics
weight: 10
---


{{< alert level="info" >}}
This action requires the following credentials:
* `user` — the username under which the action runs.
* `password` — the password for that user.
{{< /alert >}}

CreateKafkaTopics — creates new topics in Kafka.

### Request example

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

### Request specification

| Name                       | Required | Description                                                                                                                                                                                                                        | Possible values                                                                                    |
| -------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| securityProtocol           | Yes      | Protocol for connecting to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                               | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                                                                |
| saslMechanism              | No       | SASL authentication mechanism. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism)                   | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                                                                |
| partitions                 | Yes      | Number of partitions into which each topic will be divided                                                                                                                                                                         | -                                                                                                  |
| replication_factor         | Yes      | Number of copies (replicas) of each topic partition to place on different brokers                                                                                                                                                  | -                                                                                                  |
| configs                    | Yes      | Key-value configuration for the topics to create                                                                                                                                                                                   | Values are listed [in the Kafka documentation](https://kafka.apache.org/documentation/#topicconfigs) |
| topics                     | Yes      | List of topic names to create                                                                                                                                                                                                      | -                                                                                                  |
