---
title: Technology radar
description: Visualize technologies and practices by quadrant and maturity ring.
weight: 160
---

The widget visualizes technologies, tools, and practices used in the company, grouped by maturity level: Adopt, Trial, Assess, and Hold.

The widget displays a circular chart with four quadrants, four rings, and a set of items. Each item has a number, name, quadrant, and ring. Configure the radar contents in the widget settings.

## Configuration

| Name      | Required | Description                                     |
| --------- | -------- | ----------------------------------------------- |
| Quadrants | Yes      | Quadrant names                                  |
| Items     | No       | List and configuration of up to 200 radar items |

### Item configuration

| Name        | Required | Description                                                         |
| ----------- | -------- | ------------------------------------------------------------------- |
| Name        | Yes      | Item name                                                           |
| Number      | Yes      | Integer from 0 to 9999                                              |
| Description | No       | Item description, displayed as Markdown in the details dialog       |
| Quadrant    | Yes      | Quadrant containing the item                                        |
| Ring        | Yes      | Ring containing the item: Adopt, Trial, Assess, or Hold             |
