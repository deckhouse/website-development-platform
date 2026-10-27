---
title: Agent skills
description: Creating a skill with SKILL.md, publishing versions, binding skills to agents and workspaces, and installing them in an IDE.
weight: 65
---

A skill is a directory with instructions in `SKILL.md` and additional scripts, reference files, and materials.
The portal stores skill versions and passes them to agents and workspaces.

A skill defines how to perform a task.
Access to tools and objects is still defined by MCP collections and user roles.

## Creating and publishing

To create a skill for preparing project changes, follow these steps:

1. Open "AI" → "Skills" → "Skills" and click "Create skill".
1. Set the name `project-check`, a description, the owner, and the team.
1. Open the draft on the "Files" tab and fill in `SKILL.md`:

   ```markdown
   ---
   name: project-check
   description: Checks project changes before submitting them for review. Use after changing project files.
   metadata:
     version: "1.0"
   ---

   # Checking changes

   1. Read README.md and the project instructions.
   2. Identify the tests related to the changed files.
   3. Run these tests and check the result.
   4. Show the list of changes, the checks performed, and the remaining limitations.
   ```

1. If needed, add files with the "Add file" button: scripts to `scripts/`, reference materials to `references/`, and other materials to `assets/`.
1. Save the draft and check the errors and warnings panel.
1. Click "Publish version", set a unique label, for example, `v1`, and publish.

The skill name and description are defined by the `name` and `description` fields in `SKILL.md`.
To change them after creation, publish a new version.
Published files are immutable.

## Importing an archive

For import, prepare a `tar.gz` or `zip` archive with a single root directory whose name matches the `name` in `SKILL.md`:

```text
project-check/
  SKILL.md
  references/
    checklist.md
  scripts/
    check.sh
```

In the skills section, select "Upload from archive", upload the file, and check the skill contents and versions.
Allowed sizes are listed in [Limits](#limits).

## Binding to an agent

To let an agent receive a published skill, follow these steps:

1. Open the agent in "AI" → "Agents" → "Catalog" and select editing.
1. On the "Tools and skills" tab, add the skill and select a specific version or "Latest".
1. In "MCP collections", attach a collection with the `get_skill` tool; to read additional files, also add `get_skill_file`.
1. Grant the running user access to these tools through the MCP collections available to them.
1. Save the agent and start a new run.

At run time, the agent receives the names and descriptions of the assigned skills and loads the full text when needed.
Without an available `get_skill`, the skill set is not passed; the bindings form shows a warning.
`list_skills` is intended for a regular MCP client; the agent receives its own prepared set.

The "Latest" binding selects the active version with the highest number at the next run.
A pinned binding keeps the selected version.

## Binding to a workspace

Open a [workspace template](../workspaces/templates/), go to the "Skills" tab, and add the required skills.
For each binding, select a version or "Latest".

The set is fixed when the workspace is created and every time it starts.
A new skill version does not immediately replace the set in a running workspace; stop and start it again.
The workspace card shows the fixed skills and available updates.

For a template with Git, configure "Allow repository skills".
When the switch is off, the "Hidden skill folders" field sets the paths excluded from the working copy.
For example, `.agents/skills` or `**/skills`.

Hiding uses Git sparse-checkout and does not delete files from the repository.
Local changes, untracked files, and conflicts may remain on disk; the list of such files appears as a warning in the startup log.
The repository contents remain accessible through `git show`.

## Installing in a local IDE

To use a skill outside a Deckhouse Development Portal (DDP) workspace, open its card and go to the "Install in IDE" block:

1. Select "Latest" or "Specific version".
1. Create a portal API key in your profile and set it in the `DDP_API_KEY` environment variable of the terminal.
1. Copy the command from the card and run it in the root of the local project.
1. Check that `.agents/skills/project-check/SKILL.md` has appeared.

The command downloads the archive of the selected version and extracts it to `.agents/skills`.
Running it again may replace files of an already installed copy of the skill.

## Deprecated versions and archive

On the "Versions" tab, you can mark a version as deprecated and specify the reason.
Such a version is not selected as "Latest"; pinned bindings to it are preserved.
If no active versions remain, "Latest" bindings skip the skill.

An archived skill is skipped in new runs with a warning.
The "Usage" tab shows the agents and templates available to the user to which the skill is bound.
Check this list before archiving.

## Limits

When preparing a skill, take the following limits into account:

| Limit | Value |
| --- | --- |
| Name | Up to 64 characters: lowercase Latin letters, digits, and single hyphens between them |
| Description | Up to 1,024 characters |
| Number of files | Up to 1,000 |
| File size | Up to 10 MiB |
| Total size of the skill and the imported archive | Up to 25 MiB |
| Number of assigned skills | Up to 20 per agent or template |
| Text file in the editor and when read through MCP | Up to 1 MiB |

Scripts, `allowed-tools`, a long `SKILL.md`, and a missing `metadata.version` may produce warnings.
Check them before publishing.
