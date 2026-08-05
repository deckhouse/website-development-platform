---
title: CreateKafkaACLs
weight: 50
---


{{< alert level="info" >}}
Running this action requires credentials:
* `user` — the username on whose behalf the action will be run.
* `password` — the password of the user on whose behalf the action will be run.
{{< /alert >}}

CreateKafkaACLs — creates a set of ACLs in Kafka.

### Request example

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

### Request specification

| Name                     | Required | Description                                                                                                                                                                                                                          | Possible values                                          | Default value             |
| ------------------------ | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------ |
| securityProtocol         | Yes      | Protocol used to connect to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                       | -                        |
| saslMechanism            | No       | Authentication mechanism to be used by SASL. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism)      | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                       | -                        |
| acls                     | Yes      | Set of ACLs to create                                                                                                                                                                                                                | -                                                         | -                        |
| acls.ops                 | Yes      | List of operations for which the rule will be created                                                                                                                                                                                | [List of possible operations](#list-of-possible-operations) | -                        |
| acls.pattern             | Yes      | Pattern type                                                                                                                                                                                                                          | [List of possible patterns](#list-of-possible-patterns)  | -                        |
| acls.topics              | No       | List of topic names to which the rule applies                                                                                                                                                                                        | -                                                         | -                        |
| acls.groups              | No       | List of group names to which the rule applies                                                                                                                                                                                        | -                                                         | -                        |
| acls.transactional_ids   | No       | List of transaction IDs to which the rule applies                                                                                                                                                                                    | -                                                         | -                        |
| acls.tokens              | No       | List of tokens to which the rule applies                                                                                                                                                                                             | -                                                         | -                        |
| acls.allow               | No       | List of principals (user, group) for which the rule is allowed                                                                                                                                                                       | -                                                         | -                        |
| acls.deny                | No       | List of principals (user, group) for which the rule is denied                                                                                                                                                                        | -                                                         | -                        |
| acls.hosts               | No       | List of hosts for which the operation is allowed                                                                                                                                                                                     | -                                                         | -                        |
| acls.deny_hosts          | No       | List of hosts for which the operation is denied                                                                                                                                                                                      | -                                                         | -                        |

### List of possible patterns

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

### List of possible operations

For more information, see [the Kafka documentation](https://kafka.apache.org).

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

Tokens:

* DESCRIBE.
