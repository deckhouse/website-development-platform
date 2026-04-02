---
title: Teams and Users
---

The platform has two built-in entities: teams and users. The primary source of authorization data is Dex, installed in the Deckhouse Kubernetes Platform. Creating built-in users and teams is not supported.

User team membership is determined based on information from Dex and is updated each time the user authenticates.

Each user can be a member of any number of teams. User team membership is determined based on groups in Dex.

## Synchronization

When a user first logs in to the platform, an internal account is created for them. User teams are synced upon their first and each subsequent login.

### Filtering Groups During Synchronization

By default, all user groups are included in DDP during synchronization. To limit the set of teams synchronized, you can configure filtering rules.

This setting is available in the "Administration" → "Teams" section by clicking the "Synchronization Rules" button. Configuring rules requires the `edit:team-filter-rules` permission.

Rules are applied sequentially, in a chain: each rule filters the result of the previous one. An empty rules list means no filtering—all groups are synchronized.

**Rule modes:**

- **Include** — keep only teams that match the pattern. All others are excluded.
- **Exclude** — keep only teams that do not match the pattern. All matches are excluded.

**Pattern** is specified as a regular expression (regex). For example:

- `^dev-.*` — groups starting with `dev-`;
- `^team-(frontend|backend)$` — groups `team-frontend` and `team-backend`;
- `.*-prod$` — groups ending with `-prod`.

The order of rules can be changed by dragging and dropping. An empty pattern means the rule is not applied (skipped).

{{< alert level="info" >}}
If a user was part of a team and the team was subsequently filtered by rules, the user will automatically be removed from that team upon their next login. The team itself will remain in the system and must be manually deleted in "Administration" → "Teams."
{{< /alert >}}

## User Settings

### Last Activity

The user's last activity on the platform. Displayed in the user table in "Administration" → "Users." It is updated upon login or when the user's JWT token is automatically rotated (the rotation interval depends on the Dex configuration and is 10 minutes by default).

> The last activity time is not updated when using the API token issued by the user or when the user performs any actions on the platform between automatic JWT token rotations.