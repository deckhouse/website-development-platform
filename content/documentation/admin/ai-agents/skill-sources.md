---
title: Skills from GitLab
description: Preparing a skill repository, configuring a GitLab source, synchronizing versions, and archiving skills.
weight: 66
---

A skill source synchronizes directories from GitLab with the Deckhouse Development Portal (DDP) skill catalog.
Edits are made in GitLab; the portal publishes immutable versions.

## Repository structure

Place skills in the immediate subdirectories of the selected path, each with its own `SKILL.md`:

```text
skills/
  project-check/
    SKILL.md
    references/
      checklist.md
  release-prepare/
    SKILL.md
    scripts/
      check.sh
```

The skill directory name must match the `name` in its `SKILL.md`.
The file format and limits are described in the [skills guide](skills/).

## Connecting a source

For synchronization, prepare a GitLab connection and the credentials of a user with access to the project. Then follow these steps:

1. Open "AI" → "Skills" → "Skill sources" and click "Create source".
1. Set the source name, ID, owner, and team.
1. On the "Repository" tab, select "GitLab external service" and "Credentials user".
1. Specify "Project" as `group/project` or a numeric ID, "Branch or tag", and "Skills folder", for example, `skills`.
   For the repository root, use `.`.
1. On the "Synchronization" tab, set "Schedule (cron)" and turn on "Synchronize on schedule" if you need automatic synchronization.
1. Save the source and click "Sync".
1. Check "Sync history" and the created skills.

A disabled source can be synchronized with the button.
The source team manages the source itself when `owner-is-admin` is enabled; this team is not automatically assigned to the source's skills.

## Publishing changes

When the directory contents change, synchronization creates a new version labeled with the first 12 characters of the commit SHA.
If the files have not changed, no new version appears.

Agents and templates with the "Latest" binding receive the new active version at the next run.
Pinned bindings continue to use the selected version.

The sync history shows created, updated, unchanged, restored, archived, and skipped skills.
For a skipped skill, check the reason, the `SKILL.md` format, the directory name, file sizes, and the GitLab credentials.

## Removing skills from a source

If a directory disappears from the repository, the skill is archived.
Restoring the directory restores a skill archived by the source.

To delete a source, all its skills must be archived.
After deletion, they remain archived manual skills with their versions.
