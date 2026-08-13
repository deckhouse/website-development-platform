---
title: Jenkins
description: Monitor Jenkins pipelines and manage regular or multibranch builds from a dashboard.
weight: 80
---

The widget displays Jenkins pipeline data and lets you manage builds.

## Configuration

| Name | Required | Description                                                                          | Default |
| ---- | -------- | ------------------------------------------------------------------------------------ | ------- |
| URL  | Yes      | Jenkins URL used to retrieve data                                                    | —       |
| Name | Yes      | Jenkins pipeline name. Nested paths are supported: `folder1/folder2/jobName`        | —       |

## Displayed data

The widget automatically detects the pipeline type and displays the appropriate view.

### Regular pipelines

For regular pipelines, the widget displays:

* **Build list** — A table of all pipeline builds with their number, status, duration, execution time, and user.
* **Latest build** — Information about the latest completed build.
* **Latest successful build** — Information about the latest successful build.
* **Latest failed build** — Information about the latest failed build.

### Multibranch pipelines

For multibranch pipelines, the widget displays:

* **Branch list** — A table of all branches with their status, build count, and latest build.
* All information described for regular pipelines, grouped by branch.

## Additional widget features

The widget supports the following actions.

### Regular pipelines

* **Start build** — Starts a new build. If the build has parameters, the widget opens a dialog for entering:
  * String parameters.
  * Passwords.
  * List selections.
  * Boolean values.
* **Cancel build** — Cancels a running build.
* **Rebuild** — Runs the latest build again.
* **View logs** — Displays build execution logs.

### Multibranch pipelines

* **Start branch build** — Starts a new build for a specific branch. If the build has parameters, the widget opens a dialog for entering them.
* **Get branch builds** — Loads the build list for a specific branch.
* **Scan multibranch** — Starts a multibranch pipeline scan to discover new branches.
* **View logs** — Displays build execution logs.


## Authorization

Authorization is configured in [External services](../../external-services/#jenkins).
