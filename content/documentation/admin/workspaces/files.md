---
title: Template files and scripts
description: Adding files to a workspace, preserving user changes, and scripts for initial setup and start.
weight: 30
---

The starter [workspace template](templates/) delivers files from `files/` to the `/workspace/code` working directory.
Scripts from `scripts/` perform setup after the files are delivered.
The examples below modify the existing starter template.

## Adding a file to the working directory

For example, to provide editor settings to every workspace:

1. Open the template, the "Files" tab, and its draft.
1. Click "Add file" and specify the `files/.vscode/settings.json` path.
   You can also use "Upload files" for UTF-8 text files; check the path of each uploaded file.
1. Fill in the file:

   ```json
   {
     "editor.formatOnSave": true,
     "files.insertFinalNewline": true,
     "files.trimTrailingWhitespace": true
   }
   ```

1. Click "Save", then publish and activate the new version.
1. Create a workspace from this version and open the IDE.

`/workspace/code/.vscode/settings.json` appears in the working directory.
The `files/` prefix is not part of the target path.
Nested directories are created automatically.

By default, a file has the `managed` policy, and its contents are overwritten with the template version contents on every start.
This is suitable for settings managed by the administrator.
For files that the user will change, set a different policy.

## Preserving user changes in a file

To provide initial instructions to the agent and allow the owner to change them, create `files/AGENTS.md` or modify the existing file:

```markdown
# Project instructions

Read README.md before changing the project.
Run the tests related to changed files.
Show the test results and changed files in the final response.
```

In the existing `locals` block of `main.tf`, set the policy by the path relative to `files/`:

```hcl
file_policies = {
  "AGENTS.md"             = "create_if_missing"
  ".vscode/settings.json" = "create_if_missing"
}
```

The `create_if_missing` policy creates a file only if it does not yet exist in the working directory.
An existing file, including one obtained from Git, is preserved.
Changing the file in a new template version does not replace the user's copy.

The policies work as follows:

| Policy | On first start | On subsequent starts |
| --- | --- | --- |
| `managed` | Writes the contents from the template, replacing an existing file | Writes the contents from the template version in use again |
| `create_if_missing` | Creates the file only if it is missing | Preserves the existing file |

In the default template, `AGENTS.md` already has the `create_if_missing` policy.
Other files without an explicit policy use `managed`.

## Initial setup of a new volume

For example, to create a directory for results and provide an initial application configuration:

1. Add the `files/config/application.example.json` file:

   ```json
   {
     "listen": "0.0.0.0:8080",
     "logLevel": "info"
   }
   ```

1. Add the `scripts/create.sh` file:

   ```bash
   #!/usr/bin/env bash
   set -euo pipefail

   mkdir -p .local/results
   if [ ! -e config/application.json ]; then
     cp config/application.example.json config/application.json
   fi
   ```

1. Save the files, then publish and activate the version.
1. Create a workspace and check the `.local/results` directory and the `config/application.json` file.

The script runs from the working directory after the repository is cloned, files are delivered, and skills are prepared.
After a successful run, a mark is saved on the volume; subsequent starts on the same volume skip `create.sh`.
If the script fails, the next attempt runs it again, so check whether files have already been created.

Adding `create.sh` to a new template version does not repeat an initial setup that has already succeeded on an existing volume.
For setup on every start, use `start.sh`.

## Setup on every start

To check the configuration and create temporary directories after every Pod start, add `scripts/start.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

mkdir -p .local/cache
test -s config/application.json
printf 'Workspace configuration is ready\n'
```

For this example, first add `create.sh` and the initial file from the previous example.
On a successful start, the `bootstrap` container outputs `Workspace configuration is ready`.

The script runs before the agent and IDE start.
Commands must terminate; do not start a server in the script that occupies the process indefinitely.
A script error prevents the Pod from becoming ready and is shown in the `bootstrap` log on the "Diagnostics" tab.

The default Pod runs with UID/GID `10000`, without privilege escalation, and with a read-only root file system.
Place working data and installed tools on the persistent volume.
For an example of installing Go and configuring the agent and IDE environment, see [the development tools guide](tools/#installing-go).

## Applying changes to an existing workspace

For an existing workspace to receive new files:

1. Publish a new template version.
1. Stop the workspace.
1. Update it to the new version and wait for the operation to succeed.
1. Start the workspace.
1. Check the files in the IDE and the `bootstrap` container log.

For `managed`, the new contents are applied.
For `create_if_missing`, the existing file is preserved.
Activating a template version does not by itself change files of already created workspaces.
