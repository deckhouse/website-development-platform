---
title: Overview
weight: 10
---

For each resource, you can configure any number of rules that determine the status of its entities. The `Condition` field determines whether:

- all rules must pass (`AllOf`);
- at least one rule must pass (`AnyOf`).

{{< alert level="info" >}}
If at least one check returns an error, the entity status is set to `error`, regardless of the other check results and the condition.
{{< /alert >}}

## Schedule

The check scheduler runs every minute. For each rule, you can specify a five-field cron expression in the **Schedule** field. The rule then runs only at the specified times. If the field is empty, the rule runs every time the scheduler starts (once a minute).

Example: `0 * * * *` runs the check at the beginning of every hour.

If a rule is not scheduled to run at the current time, its latest check result is used to calculate the entity status.

Logs for the latest checks are available under **Health checks** in the resource menu.

## Entity statuses

An entity can have one of four statuses:

- `healthy` — rules are configured, and the entity parameters satisfy them;
- `unhealthy` — rules are configured, but the entity parameters do not satisfy them;
- `unknown` — no rules are configured, or the check cannot run;
- `error` — an error occurred while at least one rule was running.

Click an entity status badge to open a table with rule results and additional information. The table shows only rules that have run at least once.

{{< alert level="info" >}}
The `ENTITY_UPDATED` event is generated only when the entity status changes.
{{< /alert >}}
