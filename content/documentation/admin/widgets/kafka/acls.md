---
title: Kafka. ACLs
description: Access-control rule inspection and management for Kafka clusters.
weight: 10
---

The widget displays a list of access control lists (ACLs) for a Kafka cluster.

For each ACL, the widget displays:

* Principal.
* Resource type.
* Pattern.
* Pattern type.
* Host.
* Operation.
* Permission type.

## Configuration

| Name                    | Required | Description                                                                                                                                                                  | Default value |
| ----------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |
| URL                     | Yes      | Kafka cluster URL                                                                                                                                                            | —             |
| Authentication protocol | Yes      | Protocol used to connect to Kafka. [Authentication protocol reference](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                         | —             |
| SASL mechanism          | No       | Authentication mechanism used by SASL. Required when using the `SASL_PLAINTEXT` or `SASL_SSL` protocol. [SASL reference](https://kafka.apache.org/documentation/#security_sasl_mechanism) | —             |
| Kafka user              | Yes      | Username of the account used to interact with Kafka                                                                                                                          | —             |
| Password                | Yes      | Password of the account used to interact with Kafka                                                                                                                          | —             |
| Resource types          | No       | Filter by resource type                                                                                                                                                      | —             |
| Pattern types           | No       | Filter by pattern type                                                                                                                                                       | —             |
| Operations              | No       | Filter by operation                                                                                                                                                          | —             |
| Permission types        | No       | Filter by permission type                                                                                                                                                    | —             |
| Principals              | No       | Filter by principal. Supports templates and regular expressions                                                                                                              | —             |
| Hosts                   | No       | Filter by host. Supports templates and regular expressions                                                                                                                   | —             |

## Additional widget capabilities

When actions are enabled in the settings, the widget allows users to create and delete ACL rules.

## Authentication

The widget requires a user account.
The system supports the following authentication methods:

* `PLAINTEXT`.
* `SCRAM-SHA-256`.
* `SCRAM-SHA-512`.
