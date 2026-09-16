---
title: Entity table
description: Configuration of the widget that displays Deckhouse Development Portal entities in a table.
weight: 30
---

The widget displays entities created in Deckhouse Development Portal (DDP) as a table.

## Configuration

| Name         | Required | Description                                                                                                   | Default |
|--------------|----------|---------------------------------------------------------------------------------------------------------------|---------|
| Resource     | Yes      | Resource whose entities are displayed in the table                                                           | —       |
| Show actions | No       | Whether to display entity actions, such as running actions and scenarios or deleting entities                 | `false` |
