---
title: Workspaces
description: Create a workspace with an IDE and an AI agent, work with a repository, and manage and diagnose the workspace.
weight: 55
---

A workspace is a personal IDE and an OpenCode AI agent in Kubernetes. The agent reads and changes files, runs commands, and responds in the workspace chat or the AI assistant. The IDE and chat use the same files. Data remains on a persistent volume after the workspace stops.

A workspace is created from a template that defines the repository, tools, and launch parameters. Depending on the template, you can create a workspace with a repository or an empty working directory.

## Your workspaces

Open **Home** → **AI tools** → **Workspaces**. This section shows your workspaces.

Find a workspace by name or repository and select a status filter: **All**, **Running**, **Stopped**, **In progress**, or **Need attention**. The card shows the status, repository, and current operation. Click **Open** to open the agent chat page or **IDE** to open the editor. The usage of configured limits is shown above the list.

## Creating a workspace

Before creating a workspace, connect an [AI provider](ai-assistant/#connecting-an-ai-provider). If the template uses personal credentials, configure credentials for its Kubernetes and Git connections in your profile.

To create a workspace from the home page:

1. Open **Home** → **AI tools** → **Workspaces** and click **Create**.
1. In the **Template** step, select a template card and click **Next**. You can search for templates by name and description.
1. In the **Parameters and launch** step, enter a **Name** and check the template version. For a regular launch, use the active published version.
1. If the template allows repository selection, enter the **Repository URL** and, if needed, a **Branch or tag**. An empty branch selects the repository's default branch. The URL must use HTTPS and belong to the template's Git connection.
1. If needed, select **Resource of the entity** and **Catalog entity**. This binding is set at creation and allows the entity's processes to run tasks in the workspace.
1. Enter the required template parameters and select an **AI provider** and **Lifetime**. Check the summary on the right and click **Create**.
1. Wait for the **Running** status. After creation, the workspace page opens. It shows the operation log while the operation is in progress and the agent chat after the workspace starts.

If no templates are available or the creation button is unavailable, contact the portal administrator.

## Workspace page

Click **Open** on the workspace card to open the workspace page. The agent chat is in the center. The buttons in the header open the IDE and apps and start or stop the workspace. Version updates and deletion are available in the **More** menu.

Click **Details** to open a side panel with the **Operations**, **Diagnostics**, and **Apps and ports** tabs. Use them to check template application, pod status, and published apps. The chat remains on the page while the panel is open.

If the workspace is stopped, start it to continue the chat. If an operation fails, use **Show log**, resolve the cause, and click **Retry operation**. Only the workspace owner can use the agent chat.

## Working in the IDE

Open the card of a running workspace and click **Open IDE**. Allow the browser to open a new tab.

The IDE opens on a separate HTTPS domain. The portal checks access. To sign in again after the session expires, click **Open IDE**. The working directory in the standard template is `/workspace/code`.

In the IDE terminal, the `opencode` command connects to the same agent that responds in the portal chat. File changes are visible in both the IDE and the chat. To save the result in Git, review the changes, create a commit, and push it to the repository.

## Workspace apps

On the workspace page, click **Details** → **Apps and ports**. The panel shows template apps and published ports. In the starter template, the **Terminal** app opens a command line in the browser with access to workspace files. The terminal is intended for the workspace owner.

To open an app, click **Open** on its card. The **Running** indicator means that the port is listening in the pod. Check the app's readiness after opening it. After starting a server, click **Refresh** to reload the port status.

### Publishing a port

To preview a web app, publish its port:

1. Start an HTTP server in the IDE, web terminal, or through the agent. It must be accessible at `127.0.0.1` inside the pod. Port publishing supports HTTP and WebSocket.
1. Open **Details** → **Apps and ports** and click **Publish port**. You can also click **Publish** for a port under **Ports listening in the Pod**.
1. Enter the **Name**, **Port**, and **Access level**. The template limits the available access levels.
1. Confirm the publication and open the resulting address. If needed, click **Copy the address**.

The workspace owner can publish ports. Publishing creates a route; start the app server separately. For a server that checks the `Host` header, allow the external host of the published port.

In the starter template, the address has the form `https://p1-ddp-ws-<WORKSPACE_SUFFIX>.<IDE_DOMAIN>`. `<WORKSPACE_SUFFIX>` is eight hexadecimal characters from the workspace UUID, and `<IDE_DOMAIN>` is the domain set in `local.ide_domain`. The template defines the number of available addresses; the starter template reserves eight. If all reserved addresses are occupied, unpublish an unused port. If the template reserves no addresses (`app_slots = 0`), change the template settings to publish a port. A port already used by a template app cannot be published again.

### Access through a link

Before sharing a link, select an **Access level**:

| Level | Who can open it |
| --- | --- |
| **Owner only** | The workspace owner |
| **DDP users** | Any signed-in portal user who is not blocked |
| **Public** | Any visitor without signing in to the portal |

Access to an app through a link does not grant access to the workspace card, IDE, or chat. Keep web terminals available only to the owner: commands can access the owner's files and tokens inside the pod.

Change the access level of a published port in its row. To close the address and free a slot, click **Unpublish** and confirm. The server inside the pod continues to run. Route changes take approximately 15 seconds to apply.

Stopping the workspace makes its apps unavailable. Published addresses are used again after the workspace starts. If **The current template version does not reserve this address** appears after a template update, unpublish the port and publish it to an available address.

## Giving the agent a task in chat

To have the agent change a project:

1. Click **Open** on the workspace card on the home page. For an existing AI assistant chat, use **Attach a workspace**.
1. Make sure the chat shows the attached workspace.
1. Send a task with the expected result, for example:

   ```text
   Examine the project, add a /healthz check and a test for it. Run the related tests and show the changed files. Do not create a commit.
   ```

1. Review the response, commands run, and file changes.

One task runs in a workspace at a time. Subsequent messages wait in a queue. Stopping a response interrupts the current task; the workspace continues to run.

The confirmation mode controls whether the agent performs actions automatically or asks for permission. In confirmation mode, respond to the request in chat. You can revoke a saved permission for a repeated action in the workspace chat settings.

## Stopping and lifetime

To stop a workspace, click **Stop** and confirm. The IDE and current task are interrupted; files and state on the persistent volume are preserved.

**Lifetime** can be set from 1 to 720 hours. When it expires, the portal stops the workspace automatically. Stopped workspaces do not count towards the running workspace limit; their persistent volumes continue to use storage.

To resume, click **Start**. On startup, the standard template preserves the existing Git working copy and does not clone it again. Use regular Git commands to retrieve changes from the remote repository.

## Updating the template

Publishing a new active template version does not update existing workspaces automatically.

To apply a new version:

1. Stop the workspace.
1. Open **More** → **Update version**.
1. Check the selected version and parameters. If the active published template version is newer than the current one, the form selects it. The current draft remains selected if it is available to the user. If needed, select another available version. The form marks new, removed, and changed values.
1. Confirm the update and check its result on the **Operations** tab. You can switch to another version without changing parameter values.
1. Start the workspace after the update succeeds.

Data on persistent volumes is preserved. A template change that deletes or replaces an existing persistent volume is rejected. Files managed by the template may be overwritten on the next start. Before updating, save any needed changes in Git.

## Diagnostics

On the workspace page, click **Details** and find the latest failed operation. Use the appropriate tab:

| Tab | What to check |
| --- | --- |
| **Operations** | OpenTofu output during creation, startup, update, or deletion |
| **Diagnostics** | StatefulSet, pod, container, and persistent volume status; Kubernetes events; current and previous container logs |

Check task execution errors in the agent chat.

Common symptoms and actions:

| Symptom | Action |
| --- | --- |
| Cannot clone the repository | Check the Git URL and token for the account used by the template |
| The pod cannot pull an image | Send the diagnostics error message to the administrator |
| A persistent volume is not ready | Send the storage connection error from diagnostics to the administrator |
| The `bootstrap` container fails | Read its log. Check the repository and credentials; report template errors to the administrator |
| The IDE does not open while the pod is running | Click **Open IDE** again. If the error persists, send the address and browser message to the administrator |
| The running workspace limit is reached | Stop an unused workspace or ask the administrator to change the limit |

After resolving the cause, click **Retry operation**.

## Deleting a workspace

Before deletion, push any needed changes to Git or copy files from the IDE. On the workspace page, open **More** → **Delete** and confirm.

Deletion removes the workspace's Kubernetes objects and persistent data. Stop the workspace for a break; delete it to release its resources permanently.
