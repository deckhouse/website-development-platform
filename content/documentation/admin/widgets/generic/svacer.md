---
title: Svacer
description: Review static analysis findings, trends, and snapshots from Svacer.
weight: 50
---

The widget displays static code analysis results for a project branch in Svacer: marker review progress, findings by severity, branch finding trends, snapshot comparisons, and a paginated marker table.

## Configuration

| Name                        | Required | Description                                                                                                                  | Default |
| --------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------- | ------- |
| Project name                | Yes      | Project name in Svacer                                                                                                       | —       |
| Branch name                 | Yes      | Branch name in Svacer, used as the default branch when the widget opens                                                      | —       |
| Cache the full marker list  | No       | Caches the selected Svacer snapshot's full marker list in the DDP backend to speed up pagination on the **Findings** tab    | Enabled |
| Cache lifetime (seconds)    | No       | Number of seconds to keep the response in DDP memory when caching is enabled. Valid range: 30–86400                          | 180     |
| Snapshot marker threshold   | No       | Number of markers above which a snapshot is considered large                                                                | 10,000  |
| Large snapshot strategy     | No       | Behavior when the marker threshold is exceeded                                                                               | Hybrid  |

Large snapshot strategies:

* **Hybrid** — The unfiltered **Findings** tab is unavailable. The overview is generated using lightweight Svacer requests.
* **Unlimited** — The full marker list is always loaded. For a large snapshot, the widget only displays a warning.

## Query parameters

The following parameters are available when viewing the widget:

* **Branch** — Svacer branch from which data is loaded. The list is generated for the project in the widget configuration. The configured branch is selected by default.
* **Snapshot** — Branch snapshot from which data is loaded. **Latest snapshot** refers to the current branch snapshot when the widget is refreshed.
* **Filters**:
  * **Review status** — Marker review status: confirmed, false positive, unclear, no decision, or will not fix.
  * **Severity** — Finding severity.
  * **Checker** — Checker name, for example, `gosec.G402`.

## Tabs

* **Overview** — Review progress, number of unreviewed markers, branch findings, findings by severity, finding trends, reviews by status, and comparison with the previous snapshot: new, fixed, matched, and unchanged.
* **Findings** — Paginated marker table with **Severity**, **Confidence**, **Location**, **Checker**, **Review**, **Message**, and **Tags** columns. Each marker links to Svacer.

The widget footer displays summary metrics for the selected snapshot: total markers, unreviewed markers, and filtered findings by severity.

{{< alert level="info" >}}
With the **Hybrid** strategy, if the snapshot marker count exceeds the threshold, the **Findings** tab is unavailable unless you filter by review status, severity, or checker.
{{< /alert >}}

## Authorization

Authorization is configured in [External services](../../external-services/#svacer).
