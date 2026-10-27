---
title: CreateNexusUser
weight: 90
---

{{< alert level="info" >}}
Running this action requires a token: the `admin:password` string encoded in Base64.
The token is used for HTTP Basic authentication in requests to Nexus.
{{< /alert >}}

CreateNexusUser — creates a new user in Nexus Repository Manager 3.

## Request example

Example of the action request body:

```yaml
userId: example-user
firstName: First
lastName: Last
emailAddress: user@example.com
password: password
status: active
roles:
  - nx-admin
```

## Request specification

The action request body supports the following parameters:

| Name          | Required          | Description                                                                     |
| ------------- | ----------------- | ------------------------------------------------------------------------------ |
| `userId`      | Yes                | Unique user identifier                                                            |
| `firstName`   | Yes                | User's first name                                                                 |
| `lastName`    | Yes                | User's last name                                                                  |
| `emailAddress` | Yes                | User's email address                                                              |
| `password`    | Yes                | User's password                                                                   |
| `status`      | Yes                | User status: `active` or `disabled`                                              |
| `roles`       | No                 | List of role identifiers assigned to the user at creation                         |
