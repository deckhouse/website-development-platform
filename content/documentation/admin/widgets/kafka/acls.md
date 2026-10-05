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
| Authentication protocol | No       | Protocol used to connect to Kafka. [Authentication protocol reference](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                         | `PLAINTEXT`   |
| SASL mechanism          | No       | Authentication mechanism used by SASL. Required when using the `SASL_PLAINTEXT` or `SASL_SSL` protocol. [SASL reference](https://kafka.apache.org/documentation/#security_sasl_mechanism) | `PLAIN`       |
| User                    | No       | Username of the account used to interact with Kafka. Needed for the `SASL_PLAINTEXT` and `SASL_SSL` protocols | —             |
| Password                | No       | Password of the account used to interact with Kafka. Needed for the `SASL_PLAINTEXT` and `SASL_SSL` protocols | —             |
| Enable widget actions   | No       | Shows the widget actions: creating and deleting ACL rules | `true`        |
| Resource types          | No       | Filter by resource type                                                                                                                                                      | —             |
| Pattern types           | No       | Filter by pattern type                                                                                                                                                       | —             |
| Operations              | No       | Filter by operation                                                                                                                                                          | —             |
| Permission types        | No       | Filter by permission type                                                                                                                                                    | —             |
| Principals              | No       | Filter by principal. Supports templates and regular expressions                                                                                                              | —             |
| Hosts                   | No       | Filter by host. Supports templates and regular expressions                                                                                                                   | —             |

## Additional widget capabilities

When **Enable widget actions** is on, the widget allows users to create and delete ACL rules.

## Authentication

The widget connects to Kafka with the account set in the **User** and **Password** fields. These fields are needed for the `SASL_PLAINTEXT` and `SASL_SSL` protocols.

The following authentication protocols are supported:

* `PLAINTEXT`;
* `SASL_PLAINTEXT`;
* `SASL_SSL`.

The following SASL mechanisms are supported for the `SASL_PLAINTEXT` and `SASL_SSL` protocols:

* `PLAIN`;
* `SCRAM-SHA-256`;
* `SCRAM-SHA-512`.
