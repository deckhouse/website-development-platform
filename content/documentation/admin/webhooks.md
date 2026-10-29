---
title: Webhooks
menuTitle: Webhooks
description: Receiving data from external systems using the push model and processing it according to webhook rules.
weight: 71
---

Webhooks work similarly to datasources.
However, a datasource uses the pull model, that is, it polls infrastructure systems itself and tries to load data from them, while webhooks expect data to be sent from outside, that is, they use the push model.

Processing rules (creation, matching, update, and relation creation) are configured the same way as similar rules in datasources.

Each webhook has a unique URL built using the following template: `<BACKEND_URL>/webhooks/<WEBHOOK_UUID>`.
A webhook accepts POST requests with an array of data in any format, which is processed according to the webhook rules.
