---
title: Svacer
description: Review static analysis findings, trends, and snapshots from Svacer.
weight: 50
---

The widget displays static code analysis results for a project branch in Svacer: marker review progress, findings by severity, branch finding trends, snapshot comparisons, and a paginated marker table.

## Configuration

When adding the widget, specify the following parameters:

| Name                        | Required | Description                                                                                                                  | Default value |
| --------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------- | ------- |
| Project name                | Yes      | Project name in Svacer                                                                                                       | —       |
| Branch name                 | Yes      | Branch name in Svacer, used as the default branch when the widget opens                                                      | —       |
| Cache full marker list      | No       | Caches the selected Svacer snapshot's full marker list on the Deckhouse Development Portal (DDP) backend to speed up pagination on the **Findings** tab    | Enabled |
| Cache TTL (seconds)         | No       | Number of seconds to keep the response in DDP memory when caching is enabled. Valid range: 30–86400                          | 180     |
| Snapshot marker threshold   | No       | Number of markers above which a snapshot is considered large                                                                | 10,000  |
| Large snapshot strategy     | No       | Behavior when the marker threshold is exceeded                                                                               | Hybrid  |

Two strategies are available for large snapshots.
With **Hybrid**, the unfiltered **Findings** tab is unavailable, and the overview is generated using lightweight Svacer requests.
With **Unrestricted**, the full marker list is always loaded, and for a large snapshot the widget only displays a warning.

## Query parameters

The following parameters are available when viewing the widget:

* **Branch**: Svacer branch from which data is loaded.
  The list is generated for the project in the widget configuration.
  The configured branch is selected by default.
* **Snapshot**: Branch snapshot from which data is loaded.
  **Last snapshot** refers to the current branch snapshot when the widget is refreshed.
* **Filters**:
  * **Review status**: Marker review status: confirmed, false positive, unclear, no decision, or will not fix.
  * **Severity**: Finding severity.
  * **Checker**: Checker name, for example, `gosec.G402`.

## Tabs

The **Overview** tab displays review progress, number of unreviewed markers, branch findings, findings by severity, finding trends, reviews by status,
and comparison with the previous snapshot: new, fixed, matched, and unchanged.

The **Findings** tab displays a paginated marker table with **Severity**, **Confidence**, **Location**, **Checker**, **Review**, **Message**, and **Tags** columns.
Each marker links to Svacer.

The widget footer displays summary metrics for the selected snapshot: total markers, unreviewed markers, and filtered findings by severity.

{{< alert level="info" >}}
With the **Hybrid** strategy, if the snapshot marker count exceeds the threshold, the **Findings** tab is unavailable unless you filter by review status, severity, or checker.
{{< /alert >}}

## Authorization

Authorization is configured in [External services](../../external-services/#svacer).
