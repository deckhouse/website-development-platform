---
title: Microservice from creation to deployment
weight: 42
description: End-to-end developer workflow on the DDP Home page — creating a microservice from a template, creating an environment, deploying, ordering and connecting a database, and upgrading from a template.
---

This guide describes the path of a new microservice on the Home page: creating it from a template, deploying it to a temporary environment, ordering a PostgreSQL database, and connecting the database to the microservice.
A developer performs the scenario on the Home page without using the catalog or administration.

## Before you begin

To follow this guide, you need the following:

- A Home page configured by the portal administrator, for example, following the [Quick Home page setup](home-quick-start/) guide.
- Access to catalog entities and permission to run processes.
- Kubernetes, Vault, and GitLab credentials in the user profile.

## Creating a microservice

Create a microservice from a template:

1. Open the Home page → "Create microservice".
1. At the "Template" step, select a template, for example, "Go service", and click "Next".
1. At the "Service" step, fill in the fields:
   - "Microservice name": `Billing API`.
   - "Microservice identifier": Filled in as `billing-api`.
   - "System": The system that the microservice will belong to.
   - "Version control namespace": The GitLab group for the repository.
1. Click "Next", fill in the template parameters, and click "Create microservice".
1. In the "Run process" dialog, click "Run".

The creation progress is shown in the "Processes running" bar.
After the process completes, the microservice appears in the "Microservices" section, and the `billing-api` repository appears in GitLab.

## Creating an environment

Create a temporary environment to test the microservice:

1. Open the Home page → "Environments" and click "Create environment".
1. Fill in "Identifier", for example, `billing-review`, and select "7 days" for "TTL".
1. Click "Run".

While the environment is being created, the table shows the `billing-review` row labeled "creating".
The environment appears in the table after the catalog is synchronized with the cluster.

To give the microservice an address in the environment, set the environment domain.
To do this, open the environment card, click the pencil icon in the "Domain" row, and enter a domain, for example, `review.example.com`.

## Deploying the microservice

Deploy the microservice to the created environment:

1. Open the Home page → "System deployment", select the microservice's system and the `billing-review` environment.
1. Click "Deploy microservice".
1. Select the `Billing API` microservice and the `billing-review` environment, and click "Next".
1. In the "Run process" dialog, click "Run".

The "Deployments" column shows the `Billing API` card labeled "creating", and after the process completes, a deployment card with replica and resource properties.
If the environment domain is set, the process from the dataset creates an Ingress with the `billing-api.review.example.com` host, which is shown in the "Infrastructure" column.

## Ordering a database

Order a PostgreSQL database for the microservice:

1. In the "System deployment" section, click "Order resource" under the "Data resources" column.
1. In the "Order resource" dialog, select the PostgreSQL ordering process in "Service". The `billing-review` environment is already selected.
1. Click "Next", and in the "Run process" dialog, click "Run".

A resource card appears in the "Data resources" column. The database connection details are written to Vault.

## Connecting the database

Connect the database to the microservice deployment:

1. Click the dot on the left side of the database card and drag a line to the `Billing API` deployment card.
1. In the "Run process" dialog, click "Run".

After the process completes, the diagram shows the connection line, and the database secrets are delivered to the application.

## Upgrading from a template

When the template author releases a new version, the "Version {version} is available" block appears on the microservice card.

Upgrade the microservice:

1. Open the Home page → "Microservices" and select `Billing API`.
1. Click "Upgrade", and in the "Run process" dialog, click "Run".
1. Open the `Upgrade <TEMPLATE_ID> to <VERSION>` merge request in GitLab, review the changes, and merge it.

## Deleting

Deckhouse Development Portal (DDP) writes the TTL to the namespace's `ddp.deckhouse.io/deletion-ttl` label.
The environment is deleted when the TTL expires if namespace cleanup by this label is configured in the cluster.
To delete the environment immediately, open its card in the "Environments" section and click "Delete environment" in the "Danger zone" block.

Deleting an environment deletes all deployments in it.
