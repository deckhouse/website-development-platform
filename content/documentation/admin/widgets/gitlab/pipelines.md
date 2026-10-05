---
title: GitLab. Pipelines
description: Viewing GitLab project pipelines, their stages, jobs and logs, and running, rerunning and canceling pipelines and jobs.
weight: 50
---

The widget displays GitLab project pipelines with their stages and jobs and lets you run, rerun and cancel pipelines and jobs from Deckhouse Development Portal (DDP).

## Configuration

| Name       | Required | Description                                                              | Default value |
| ---------- | -------- | ------------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                         | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Pipeline table

The table contains the following columns:

- **ID** — pipeline number with a link to GitLab;
- **Status** — pipeline state;
- **Ref** — branch or tag;
- **Source** — event that started the pipeline;
- **Initiator** — user who started the pipeline;
- **Duration** and **Finished**;
- the **Actions** menu.

The **Actions** menu contains:

- **Rerun pipeline**;
- **Cancel pipeline** — for pending and running pipelines;
- **Open in GitLab**.

## Stages and jobs

Click a pipeline row to expand the creation, start and finish times and the pipeline stages. For each stage, the widget displays the status, the number of failed jobs and the stage jobs. If a stage has more than eight jobs, the rest are hidden: to show them, click **Show N more**, where N is the number of hidden jobs.

Click a job to open the job panel. The panel displays the job name, the job number with a link to GitLab, the stage, the duration and the following buttons:

- **Run job** — for manual jobs;
- **Rerun job** — unavailable for manual and pending jobs;
- **Cancel job** — for pending, scheduled and running jobs.

### Job log

The job panel displays the job log: the last 50 lines by default. The **Full log** button shows the entire log, the **Last lines** button returns the shortened view. The job output colors are preserved.

If the job has not started yet, the widget displays the message "The job has not started yet — no log".

## Running a pipeline

The **Run pipeline** button in the widget header starts a pipeline in GitLab.

Run parameters:

| Name      | Required | Description                                                  | Default value |
| --------- | -------- | ------------------------------------------------------------ | ------------- |
| Ref       | Yes      | Target branch or tag on which to start the pipeline          | —             |
| Variables | No       | Key-value variables to pass to the pipeline being started    | —             |

## Home page placement

The widget is displayed on the **Pipelines** tab of a microservice card on the home page if it is selected in the `microservice_repository_widget` slot of the mapping rules.
In this case, set the **Project ID** field to `{{ .context.repository.id }}`: the microservice card passes the project ID of its repository to the widget.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.
