---
title: Development tools
description: Installing Go in a workspace with a template script, configuring PATH for the agent and IDE, and preserving tools across restarts.
weight: 35
---

You can install development tools with template scripts on the workspace persistent volume.
The agent and IDE share this volume.
An installation in `/workspace/state` persists after a stop and start, and the project files remain in `/workspace/code`.

The default containers run with UID/GID `10000` and a read-only root file system.
Select a tool distribution that can be extracted to a user directory without `sudo`, `apt`, or `apk`.

## Installing Go

The example installs Go `1.27.3` on Linux amd64 or arm64.
The version and SHA-256 checksums of the archives are fixed in the script; they are published [on the Go downloads page](https://go.dev/dl/).

### Installation script

Add the `scripts/start.sh` file with the following contents to the template draft.
If the file already contains setup commands, merge them with the example:

```bash
#!/usr/bin/env bash
set -euo pipefail

: "${DDP_STATE_DIR:?}"
go_version="1.27.3"

case "$(uname -m)" in
  x86_64)
    go_arch="amd64"
    go_checksum="65efbc80fdc1fa2cd2631cad5b481777a68e8725f34e89f50fc6ac714e379b40"
    ;;
  aarch64|arm64)
    go_arch="arm64"
    go_checksum="a06e99309dbcbdca1a8ccd2282c3ee146e55930886e26b4f1193269587165f49"
    ;;
  *)
    printf 'Unsupported architecture: %s\n' "$(uname -m)" >&2
    exit 1
    ;;
esac

toolchains_dir="${DDP_STATE_DIR}/toolchains"
go_directory="${toolchains_dir}/go${go_version}-linux-${go_arch}"
mkdir -p "$toolchains_dir" "${DDP_STATE_DIR}/tools" "${DDP_STATE_DIR}/tmp/go"

if [ ! -d "$go_directory" ]; then
  staging_dir="$(mktemp -d "${toolchains_dir}/.go-install.XXXXXX")"
  trap 'rm -rf "$staging_dir"' EXIT

  curl -fsSL --retry 3 --connect-timeout 15 --max-time 300 "https://go.dev/dl/go${go_version}.linux-${go_arch}.tar.gz" -o "${staging_dir}/go.tar.gz"
  printf '%s  go.tar.gz\n' "$go_checksum" > "${staging_dir}/go.sha256"
  (cd "$staging_dir" && sha256sum -c go.sha256)
  tar -xzf "${staging_dir}/go.tar.gz" -C "$staging_dir"
  mv "${staging_dir}/go" "$go_directory"

  rm -rf "$staging_dir"
  trap - EXIT
fi

ln -sfn "$go_directory" "${DDP_STATE_DIR}/tools/go"
"${DDP_STATE_DIR}/tools/go/bin/go" version
```

The script downloads and verifies the archive only if the selected version is missing on the volume.
The archive is extracted to a temporary directory on the same volume; the finished installation is moved to the version directory.
On subsequent starts, the saved copy is used.

The `/workspace/state/tools/go` link points to the selected version.
As a result, the path in `PATH` stays the same when the version is updated.

Use `start.sh` so that the example applies both to new workspaces and to existing ones after a template update.
`create.sh` is skipped if the initial volume setup has already succeeded.

### Agent and IDE environment

In `main.tf`, add the following fields to the existing `local.environment` block, keeping the other fields:

```hcl
PATH        = "${local.state_directory}/tools/go/bin:${local.state_directory}/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
GOPATH      = "${local.state_directory}/go"
GOCACHE     = "${local.state_directory}/cache/go-build"
GOTMPDIR    = "${local.state_directory}/tmp/go"
CGO_ENABLED = "0"
```

The default template passes `local.environment` to the initial setup container, the agent, and the IDE.
Go and commands installed with `go install` will be available in all these containers.
The build cache and `GOPATH`, including the module cache, are located on the persistent volume.
`GOTMPDIR` moves temporary compilation files to the same volume so that the build does not fill the container's limited `/tmp`.

Changing `PATH` with `export` inside `start.sh` affects only the script itself and its child processes.
Therefore, set the variables in `main.tf`.

In the example, `CGO_ENABLED = "0"` disables the use of a C compiler.
This is suitable for Go projects without C dependencies; projects with CGo also require the corresponding compiler and system libraries.

### Publishing and verification

To run the example, the Pod needs HTTPS access to `go.dev` and to the download address after the redirect, usually `dl.google.com`.
Project dependencies also require access to the selected Go module proxy or repositories.
If direct access is blocked, host the same archive in internal storage and change the URL in the script, keeping the SHA-256 check.

To apply the example:

1. Save `scripts/start.sh` and `main.tf`, then publish and activate the version.
1. Create a new workspace, or stop an existing one, update its template version, and start it again.
1. In the `bootstrap` container log on the "Diagnostics" tab, check that the archive verification succeeded and that the `go version` line is present.
1. Open the IDE terminal and check the installation:

   ```shell
   go version
   go env GOPATH GOCACHE GOTMPDIR CGO_ENABLED
   command -v go
   ```

1. Ask the agent in the chat to run `go version`. It must see the same version as the IDE terminal.
1. Stop and start the workspace again.
   Go must remain available; the same version is not downloaded again.

### Verifying the build

To check the compiler, create and run a program outside the project directory.
The example does not change `go.mod` or the project source files:

```shell
mkdir -p /workspace/state/checks/go
cat > /workspace/state/checks/go/hello.go <<'EOF'
package main

import "fmt"

func main() {
    fmt.Println("Go is ready")
}
EOF
go run /workspace/state/checks/go/hello.go
```

The expected output is `Go is ready`.
Then run the checks of the project itself, for example, `go test ./...` from its root directory.

## Updating the tool version

To update Go, change `go_version` and both archive checksums in `scripts/start.sh`.
Take the values [from the Go downloads page](https://go.dev/dl/), then save and publish a new template version.

Stop the workspace, update the template, and start the workspace again.
The script installs the new version in a separate directory and switches the `tools/go` link.
The previous version remains on the volume; you can delete its directory from the terminal after checking the new version.

## Installing other tools

For other tools, use the same sequence: select an archive for the container architecture and system libraries, verify its checksum, extract it to the persistent volume, and add the executables to the template environment.

Check compatibility separately in the agent and the IDE.
The default agent uses Alpine Linux with musl; it supports musl-compatible or statically linked binaries.
A matching processor architecture is not enough if the tool depends on system libraries.
