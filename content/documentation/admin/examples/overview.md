---
title: Overview
description: Step-by-step examples of configuring Deckhouse Development Portal — the Home page, microservice templates, and automatic service upgrades.
weight: 10
---

This section contains step-by-step examples of configuring Deckhouse Development Portal (DDP) for common administrator tasks.
Each example covers the full scenario, from preparation to verifying the result.

## Configuring the Home page

These examples show how to prepare the Home page to work with the catalog and processes:

| Example | Description |
| --- | --- |
| [Quick Home page setup](home-quick-start/) | The initial setup wizard, credentials, template registry, and process binding on a new installation |
| [Home page for a custom catalog model](home-custom-catalog/) | Mapping existing resources, relations, and properties to Home page roles without the "Canonical architecture" dataset |
| [Custom resource ordering process](home-custom-process/) | Processes for ordering and deleting a shared resource, parameters from the run context, and slot categories |
| [Widget on a microservice tab](home-widget-slot/) | Displaying a GitLab widget on microservice card tabs using widget slots |

## Microservice templates

These examples describe the lifecycle of a template and the microservices created from it:

| Example | Description |
| --- | --- |
| [Creating and publishing a microservice template](template-first-pack/) | A template pack for a Go service, validation with the `ddp-template` utility, publishing, and creating a microservice |
| [Releasing a new template version](template-release-update/) | Releasing a new version, manual and automatic microservice upgrades, and resolving conflicts |
| [Microservice from creation to deployment](microservice-lifecycle/) | An end-to-end developer scenario: creation, environment, deployment, database, and upgrade |

## Automation based on actions and processes

The [Automatic service upgrades](service-autoupdates/) example shows how to configure resources, a datasource, actions, a process, and an automation without microservice templates.
As a result, when a new version of a template repository is released, merge requests are created in the services.
