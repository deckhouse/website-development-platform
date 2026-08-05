---
title: DeleteKafkaACLs
weight: 60
---


{{< alert level="info" >}}
Running this action requires credentials:
* `user` — the username on whose behalf the action will be run.
* `password` — the password of the user on whose behalf the action will be run.
{{< /alert >}}

DeleteKafkaACLs — deletes a set of ACLs in Kafka.

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
    allow_hosts:
      - 127.0.0.1
    deny:
      - User:principal_4
      - Group:principal_5
    deny_hosts:
      - 127.0.0.1
    ops:
      - CREATE
      - READ
      - WRITE
      - DELETE
      - DESCRIBE
      - DESCRIBE_CONFIGS
      - ALTER
    pattern: LITERAL
  - any_topic: true
    any_group: true
    any_transactional_id: true
    any_allow: true
    any_allow_hosts: true
    any_deny: true
    any_deny_hosts: true
    ops:
      - ANY
    pattern: ANY
  - any_resource: true
    any_allow: true
    any_allow_hosts: true
    any_deny: true
    any_deny_hosts: true
    ops:
      - ANY
    pattern: ANY
```

### Request specification

| Name                        | Required | Description                                                                                                                                                                                                                          | Possible values                                          | Default value             |
| --------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------ |
| securityProtocol            | Yes      | Protocol used to connect to Kafka. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#adminclientconfigs_security.protocol)                                                                | PLAINTEXT, SASL_PLAINTEXT, SASL_SSL                       | -                        |
| saslMechanism               | No       | Authentication mechanism to be used by SASL. Required when using the SASL_PLAINTEXT or SASL_SSL protocol. For more information, see [the Kafka documentation](https://kafka.apache.org/documentation/#security_sasl_mechanism)      | PLAIN, SCRAM-SHA-256, SCRAM-SHA-512                       | -                        |
| acls                        | Yes      | Set of ACLs to delete                                                                                                                                                                                                                | -                                                         | -                        |
| acls.ops                    | Yes      | List of operations for which the rule will be deleted                                                                                                                                                                                | [List of possible operations](#list-of-possible-operations) | -                        |
| acls.pattern                | Yes      | Pattern type                                                                                                                                                                                                                          | [List of possible patterns](#list-of-possible-patterns)  | -                        |
| acls.any_resource           | No       | All resources                                                                                                                                                                                                                         | -                                                         | false                    |
| acls.topics                 | No       | List of topic names to which the rule applies                                                                                                                                                                                        | -                                                         | -                        |
| acls.any_topic              | No       | All topics                                                                                                                                                                                                                            | -                                                         | false                    |
| acls.groups                 | No       | List of group names to which the rule applies                                                                                                                                                                                        | -                                                         | -                        |
| acls.any_group              | No       | All groups                                                                                                                                                                                                                            | -                                                         | false                    |
| acls.transactional_ids      | No       | List of transaction IDs to which the rule applies                                                                                                                                                                                    | -                                                         | -                        |
| acls.any_transactional_id   | No       | Any transaction                                                                                                                                                                                                                       | -                                                         | false                    |
| acls.tokens                 | No       | List of tokens to which the rule applies                                                                                                                                                                                             | -                                                         | -                        |
| acls.any_token              | No       | All tokens                                                                                                                                                                                                                            | -                                                         | false                    |
| acls.allow                  | No       | List of principals (user, group) for which the rule is allowed                                                                                                                                                                       | -                                                         | -                        |
| acls.any_allow              | No       | Any principal (user, group)                                                                                                                                                                                                          | -                                                         | false                    |
| acls.allow_hosts            | No       | List of hosts for which the operation is allowed                                                                                                                                                                                     | -                                                         | -                        |
| acls.any_allow_hosts        | No       | Any host from which the operation is allowed                                                                                                                                                                                         | -                                                         | false                    |
| acls.deny                   | No       | List of principals (user, group) for which the rule will be denied                                                                                                                                                                   | -                                                         | -                        |
| acls.any_deny               | No       | Any principal (user, group)                                                                                                                                                                                                          | -                                                         | false                    |
| acls.deny_hosts             | No       | List of hosts for which the operation will be denied                                                                                                                                                                                 | -                                                         | -                        |
| acls.any_deny_hosts         | No       | Any host from which the operation is denied                                                                                                                                                                                          | -                                                         | false                    |

### List of possible patterns

* ANY.
* MATCH.
* LITERAL.
* PREFIXED.

### List of possible operations

For a detailed description, see [the Kafka documentation](https://kafka.apache.org/39/documentation/#operations_resources_and_protocols).

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

Token:

* DESCRIBE.
