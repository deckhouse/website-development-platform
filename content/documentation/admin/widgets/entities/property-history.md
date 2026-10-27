---
title: Property history
description: Chart of entity property changes over the selected period.
weight: 60
---

The widget plots a chart of entity property changes over the selected period.
The period is selected in the widget header.

## Configuration

When adding the widget, specify the following parameters:

| Name                 | Required | Description                                                                                                                                                                 | Default value  |
|----------------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------|
| Resource             | No       | Resource whose schema provides the properties. If no resource is specified, properties are set by identifiers, and the chart is plotted for the resource of the current entity | —              |
| Properties           | Yes      | Properties whose changes the chart displays. Supported types: `Number`, `Percentage`, and `Boolean`. Multiple properties must be of the same type                          | —              |
| Default interval     | No       | Period for which the widget plots the chart when opened                                                                                                                      | Last 30 days   |
| Threshold            | No       | Horizontal line on the chart, drawn only for numeric properties                                                                                                              | —              |
| Axis minimum         | No       | Lower bound of the value axis                                                                                                                                                | Based on data  |
| Axis maximum         | No       | Upper bound of the value axis                                                                                                                                                | Based on data  |

## Displayed data

The widget displays the following data:

- "Chart": Numeric properties are displayed as a value line, flags are displayed on an axis with the "No" and "Yes" values.
- "Chart point": A property change. The tooltip shows the value, the change source, and the author.
- "Legend": Displayed if the chart contains more than one property.
- "Retention boundary": A vertical mark on the chart; changes to the left of it have already been deleted.

## Notes

Property change history is recorded only when ClickHouse is connected.
If the storage is not configured, the widget reports that property history is not recorded.

The history retention period is limited.
If the entire selected period is older than the retention period, the widget reports that changes for it have already been deleted.

If the selected period contains more changes than fit on the chart, the widget displays a warning and lists the properties for which some changes are not shown.
Reduce the period or keep fewer properties on the chart.

If a non-numeric value, such as text, is written to a numeric property, it cannot be displayed on the axis.
At this point, the line breaks and continues from the next numeric value, and the value itself is visible in the tooltip.
