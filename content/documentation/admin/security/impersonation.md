---
title: Impersonation
description: Start, end, restrict, and audit temporary sessions performed as another DDP user.
weight: 25
---

DDP supports impersonation: a user with the global `impersonate:users` permission can temporarily use the web interface as another user.

{{< alert level="info" >}}
Impersonation is available only for browser sign-in through a Dex session. API tokens are not supported.
{{< /alert >}}

## Starting and ending impersonation

1. Go to "Administration" → "Users".
1. In the target user's row, click "Sign in as this user".
1. The platform switches the session to the selected user. A "Signed in as: …" banner at the bottom of the screen shows the time remaining before the session ends automatically.
1. To end impersonation early, click "End session" in the banner.

The "Sign in as this user" button is unavailable for blocked users and your own account.

## Security restrictions

Impersonation does not start in the following cases:

- The selected user is blocked.
- The selected user is the user who is already signed in.
- The selected user is a super administrator, but the operator is not a super administrator.
- The selected user already has the global `impersonate:users` permission, unless the operator is a super administrator.

## Session lifetime

An impersonation session has a limited lifetime:

- The session lasts 30 minutes.
- The session ends automatically when it expires.
- The web interface displays a countdown until automatic termination.

If the user does not end impersonation manually, the platform ends it automatically when the session expires.

## Audit

For HTTP requests included in audit logs (`POST`, `PUT`, `DELETE`, and `PATCH`), the "Email" field identifies the operator and target user in the format `operator@example.com [as target@example.com]`.

This format lets you determine the following from an audit log entry:

- Who initiated the request.
- Whose identity the session used to perform the operation.
- The HTTP method, path, and response status.

`GET` requests are not recorded in audit logs. For details, see [Audit logs](../audit-logs/).
