---
title: GitLab. Merge requests
description: Viewing, filtering, creating and merging GitLab project merge requests.
weight: 20
---

The widget displays GitLab project merge requests (MRs) and lets you create, merge and close them.

## Configuration

| Name       | Required | Description                                                              | Default value |
| ---------- | -------- | ------------------------------------------------------------------------ | ------------- |
| URL        | Yes      | GitLab API URL used to retrieve data from GitLab                         | —             |
| Project ID | Yes      | ID of the project from which the widget retrieves data. Example: `12345` | —             |

## Filtering

Above the table, there are MR state filters with the number of MRs in each state: **All**, **Open**, **Drafts**, **Merged**, **Closed**.

The **Open** filter is selected by default: it shows open MRs except drafts. Open drafts are shown in the **Drafts** filter.

The **in** list with the **All branches** option filters MRs by target branch. The number of found MRs out of the total number is displayed on the right.

## Table

The table contains the following columns:

- **ID** — MR number with a link to GitLab;
- **State** — MR state;
- **Title** — MR title, the **Draft** tag and a conflict icon with the "Conflicts with the target branch" tooltip;
- **Author** and **Reviewers**;
- **Branch** — source branch; the tooltip shows the source and target branches;
- **Updated** — time of the last change;
- the **Actions** menu.

Click a row to expand the MR details: creation and update dates, reviewers, the number of changed files with the number of added and deleted lines, the pipeline status and the **Open in GitLab** link.

## MR actions

MR actions are available in the expanded row and in the **Actions** menu:

- **Merge** — merges the MR after confirmation;
- **Close** — closes the MR;
- **Mark as draft** and **Mark as ready** — change the draft flag;
- **Diff** — the list of changed files in the expanded row, or a line-by-line file comparison in a separate window from the **Actions** menu;
- **Open in GitLab** — opens the MR in GitLab.

If an MR cannot be merged, the reason is displayed next to the buttons:

- "Only an open merge request can be merged";
- "A draft cannot be merged";
- "Conflicts with the target branch";
- "The pipeline is not green" — the latest MR pipeline has not succeeded.

{{< alert level="info" >}}
Actions on MRs require the corresponding access permissions in the GitLab repository.
{{< /alert >}}

### Creating an MR

The **Create MR** button in the widget header opens the MR creation window. Fill in the **Source branch**, **Target branch**, **Title** and, if needed, **Description** fields and click **Create**.

The source and target branches must differ. The **Title** field is required.

## Home page placement

The widget is displayed on the **MRs** tab of a microservice card on the home page if it is selected in the `microservice_merge_requests_widget` slot of the mapping rules.
In this case, set the **Project ID** field to `{{ .context.repository.id }}`: the microservice card passes the project ID of its repository to the widget.

## Authentication

Authentication configuration is described in the [External services](../../external-services/#gitlab) section.
