---
title: Kafka. Topics
description: Topic inspection and management capabilities for Kafka clusters.
weight: 20
---

The widget displays Kafka topic data.

The following information and actions are available for each topic:

* General topic information, including key parameters, configuration, and status.
* Partition information, including leaders, offsets, and replica counts.
* Consumer information, including active consumers, their groups, current offsets, and lag.
* Message content.
* Message search by timestamp and offset.
* Topic configuration as a key-value table.

## Configuration

| Name                    | Required | Description                                                                                                                                                                  | Default value |
| ----------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |
| URL                     | Yes      | Kafka cluster URL                                                                                                                                                            | —             |
| Authentication protocol | No       | Protocol used to connect to Kafka. [Authentication protocol reference](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                         | `PLAINTEXT`   |
| SASL mechanism          | No       | Authentication mechanism used by SASL. Required when using the `SASL_PLAINTEXT` or `SASL_SSL` protocol. [SASL reference](https://kafka.apache.org/documentation/#security_sasl_mechanism) | `PLAIN`       |
| User                    | No       | Username of the account used to interact with Kafka. Needed for the `SASL_PLAINTEXT` and `SASL_SSL` protocols | —             |
| Password                | No       | Password of the account used to interact with Kafka. Needed for the `SASL_PLAINTEXT` and `SASL_SSL` protocols | —             |
| Enable widget actions   | No       | Shows the widget actions: creating and deleting topics, sending messages and purging a topic | `true`        |
| Kafka topics            | No       | Topic name or regular expression used to filter topics displayed in the widget. If empty, all topics available to the user are displayed                                    | —             |

## Additional widget capabilities

When **Enable widget actions** is on, the widget allows users to:

* Create topics.
* Delete topics.
* Send a message to a topic.
* Remove all messages from a topic.

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

{{< alert level="info" >}}
The connected account's permissions determine which information is available in the widget.
{{< /alert >}}
