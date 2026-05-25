# MirthSync for VS Code

**Mirth Connect & Open Integration Engine development in VS Code.** Channel sync, IntelliSense for Mirth scripting APIs, multi-server connection management, and a one-command Local Mirth Docker stack for offline testing — no Java install, no admin rights.

[![Version](https://img.shields.io/visual-studio-marketplace/v/SagaITLLC.mirthsync?label=marketplace)](https://marketplace.visualstudio.com/items?itemName=SagaITLLC.mirthsync)
[![Installs](https://img.shields.io/visual-studio-marketplace/i/SagaITLLC.mirthsync)](https://marketplace.visualstudio.com/items?itemName=SagaITLLC.mirthsync)
[![Rating](https://img.shields.io/visual-studio-marketplace/r/SagaITLLC.mirthsync)](https://marketplace.visualstudio.com/items?itemName=SagaITLLC.mirthsync)

**By [Saga IT, LLC](https://saga-it.com)** · [Docs](https://saga-it.com/docs/mirthsync/vscode/) · [Mirth Connect Docker Quickstart](https://saga-it.com/blog/mirth-connect-quickstart)

## What's new in v2

- 🐳 **Local Mirth** — `MirthSync: Initialize Local Mirth` scaffolds a Docker Compose stack (Mirth + Postgres + tools) into your workspace and auto-wires a connection. Install Mirth Connect in Docker without touching your host. → [Quickstart](https://saga-it.com/blog/mirth-connect-quickstart)
- ⚙️ **Auto port fallback** — Local Mirth handles port-8443 conflicts and persists the choice in `.env`.
- 🔄 **`deleteOrphaned` setting** — opt-in destructive `Pull All` for full-mirror workflows.

## Features

- 🐳 **Local Mirth** — One command spins up Mirth Connect or OIE in Docker (Mirth + Postgres + tools). Nothing on your host beyond Docker. [Mirth Connect Docker quickstart →](https://saga-it.com/blog/mirth-connect-quickstart)

  ![Local Mirth](https://raw.githubusercontent.com/SagaHealthcareIT/mirthsync-vscode/refs/heads/main/media/local-mirth.gif)
- **Channel & Template Sync** — Pull/push individual channels, channel groups, code template libraries, or sync all at once
- **Connection Management** — Multi-server connection profiles with secure credential storage (VS Code Secrets API)
- **IntelliSense** — Autocomplete and hover documentation for Mirth JavaScript APIs

  ![IntelliSense](https://raw.githubusercontent.com/SagaHealthcareIT/mirthsync-vscode/refs/heads/main/media/intellisense.gif)
- **Status Bar Controls** — Quick toggles for ConfigMap inclusion, Force sync, and Deploy after push
- **File Explorer Integration** — Right-click on Channels/CodeTemplates folders to pull/push directly
- **Tree Views** — Browse channels and code templates hierarchically with context menu actions

## Quick Start

Two paths depending on whether you already have a Mirth server:

### Path A — You have a Mirth Connect server

1. Install the extension from the VS Code marketplace (or from VSIX)
2. Open Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
3. Run **MirthSync: Add Connection** and enter your Mirth server details
4. Set the connection as active and connect
5. Use tree views or commands to pull/push channels

![Add Connection](https://raw.githubusercontent.com/SagaHealthcareIT/mirthsync-vscode/refs/heads/main/media/add-connection.gif)

### Path B — Zero install: Local Mirth in Docker

1. Install the extension
2. Run **`MirthSync: Initialize Local Mirth`** in any workspace
3. Run **`MirthSync: Start Local Mirth`** — the auto-created *Local Mirth* connection is ready in ~30 seconds

Full walkthrough: [How to Install Mirth Connect, OIE, or BridgeLink (2026 Docker Quickstart)](https://saga-it.com/blog/mirth-connect-quickstart).

## Commands

### Connection Management

| Command | Description |
|---------|-------------|
| `MirthSync: Add Connection` | Add a new Mirth server connection |
| `MirthSync: Test Connection` | Test the selected connection |
| `MirthSync: Refresh Connections` | Refresh the connections tree view |

### MirthSync Operations

| Command | Description |
|---------|-------------|
| `MirthSync: Pull All` | Pull all channels and templates from server |
| `MirthSync: Push All` | Push all local changes to server |
| `MirthSync: Git Status` | Show git status of mirthsync workspace |
| `MirthSync: Git Diff` | Show git diff of changes |
| `MirthSync: Toggle ConfigurationMap Inclusion` | Toggle whether to include ConfigurationMap.xml |
| `MirthSync: Toggle Force Sync` | Toggle force overwrite on conflicts |
| `MirthSync: Toggle Deploy After Push` | Toggle automatic deployment after push |

### Tree View Context Menu Commands

| Command | Description |
|---------|-------------|
| `Pull Channel` | Pull a specific channel from server |
| `Push Channel` | Push a specific channel to server |
| `Pull Channel Group` | Pull all channels in a group |
| `Push Channel Group` | Push all channels in a group |
| `Pull Code Template Library` | Pull a template library from server |
| `Push Code Template Library` | Push a template library to server |
| `Pull Global Scripts` | Pull global scripts from server |
| `Push Global Scripts` | Push global scripts to server |

### Mirth CLI Operations

| Command | Description |
|---------|-------------|
| `MirthSync: Deploy Channel` | Deploy a channel on the server |
| `MirthSync: Undeploy Channel` | Undeploy a channel on the server |
| `MirthSync: Import Channel` | Import a channel from file |
| `MirthSync: Export Channel` | Export a channel to file |
| `MirthSync: Server Status` | Show Mirth server status |

### Local Mirth

| Command | Description |
|---------|-------------|
| `MirthSync: Initialize Local Mirth` | Scaffold `.mirthsync/local/` (compose stack + tools container) into the workspace |
| `MirthSync: Start Local Mirth` | Build the tools image and start Mirth + Postgres + tools containers (auto-falls back to an alternate host port if 8443 is busy) |
| `MirthSync: Stop Local Mirth` | Stop the containers without deleting state |
| `MirthSync: Reset Local Mirth (Delete State)` | Stop the stack and wipe its Postgres + appdata volumes |
| `MirthSync: Remove Local Mirth (Delete All)` | Stop the stack, delete its volumes and locally-built tools image, remove `.mirthsync/local/`, and delete the auto-created Local Mirth connection — pristine "as if it was never initialized" state |
| `MirthSync: Show Local Mirth Status` | Per-service running/health summary |
| `MirthSync: Show Local Mirth Logs` | Tail Mirth container logs into the MirthSync output channel |
| `MirthSync: Show Local Mirth Info` | URL, credentials, files location, launcher links — with copy actions |

Full reference: [Local Mirth docs on saga-it.com →](https://saga-it.com/docs/mirthsync/vscode/local-mirth/)

## Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `mirthsync.mirthsyncPath` | string | `""` | Path to mirthsync executable. Leave empty to auto-detect. |
| `mirthsync.mirthcliPath` | string | `""` | Path to Mirth CLI executable. Leave empty to auto-detect. |
| `mirthsync.defaultTimeout` | number | `30000` | Default timeout for operations in milliseconds. |
| `mirthsync.outputVerbosity` | number | `1` | Verbosity level for mirthsync output (0-5). |
| `mirthsync.autoSavePresets` | boolean | `true` | Automatically save preset after successful operations. |
| `mirthsync.ignoreCertificateWarnings` | boolean | `false` | Ignore SSL certificate warnings (development only). |
| `mirthsync.javadocsUrl` | string | `""` | URL to Mirth javadocs for API generation. |
| `mirthsync.forceSync` | boolean | `false` | Force overwrite when syncing channels/templates. |
| `mirthsync.promptForForce` | boolean | `true` | Prompt to use force option when sync fails due to conflicts. |
| `mirthsync.deleteOrphaned` | boolean | `false` | When pulling, delete local files that no longer exist on the server. **Destructive — opt-in.** Only applies to `Pull All`; scoped pulls (single channel/group/library) are unaffected. |
| `mirthsync.promptForDeleteOrphaned` | boolean | `true` | When `deleteOrphaned` is enabled, show a confirmation prompt at the start of each `Pull All`. Disable for unattended automation. |
| `mirthsync.includeConfigurationMap` | boolean | `false` | Include the Configuration Map when pulling or pushing. |
| `mirthsync.deployAfterPush` | boolean | `false` | Deploy channels immediately after pushing. |
| `mirthsync.skipDisabled` | boolean | `false` | Skip disabled channels when pushing. |
| `mirthsync.localMirth.mirthImageTag` | string | `"4.5.2-ubuntu-jre"` | Image tag for the Local Mirth server (`sagait/engine` — Saga-packaged Open Integration Engine). |
| `mirthsync.localMirth.mirthsyncVersion` | string | `"3.5.2"` | mirthsync release version baked into the Local Mirth tools image. |
| `mirthsync.localMirth.additionalPorts` | array | `[]` | Additional ports to forward from the Mirth container to localhost (e.g., MLLP/HTTP listener ports). The extension writes these to an auto-managed `docker-compose.override.yml` on Start. |

## Tree Views

The extension adds three tree views to the Explorer sidebar:

### Mirth Connections
Manage your Mirth server connections. Right-click for options:
- Connect/Disconnect
- Test Connection
- Set as Active
- Edit/Remove

### Mirth Channels
Browse channels organized by channel groups (visible when connected). Right-click to:
- Pull/Push individual channels
- Pull/Push entire channel groups

### Code Templates
Browse code template libraries (visible when connected). Right-click to:
- Pull/Push individual templates
- Pull/Push entire libraries

![Push and Pull Channels and Code Templates](https://raw.githubusercontent.com/SagaHealthcareIT/mirthsync-vscode/refs/heads/main/media/push-pull-channels-templates.gif)

## File Explorer Integration

When connected, right-click on folders in the file explorer to sync:
- `Channels/` - Pull or push channel configurations
- `CodeTemplates/` - Pull or push code template libraries
- `GlobalScripts/` - Pull or push global scripts
- `ConfigurationMap.xml` - Pull or push configuration map
- `Resources.xml` - Pull or push resources

![Push and Pull from File Explorer](https://raw.githubusercontent.com/SagaHealthcareIT/mirthsync-vscode/refs/heads/main/media/push-pull-file-explorer.gif)

## Status Bar

The status bar shows:
- **Connection status** - Current active connection and state
- **ConfigMap** - Whether Configuration Map is included (click to toggle)
- **Force** - Whether force sync is enabled (click to toggle)
- **Deploy** - Whether deploy after push is enabled (click to toggle)

![Status Bar Controls](https://raw.githubusercontent.com/SagaHealthcareIT/mirthsync-vscode/refs/heads/main/media/status-bar-controls.gif)

## Local Mirth — Mirth Connect in Docker, zero host install

For testing this workspace against a throwaway local Mirth Connect server without installing Java or the Mirth installer on your host, run **`MirthSync: Initialize Local Mirth`**. It scaffolds `.mirthsync/local/` with a Docker Compose stack:

- `mirth` — Saga-packaged Open Integration Engine (`sagait/engine`, OIE with Saga plugins preinstalled), exposed on `127.0.0.1:8443` (or an auto-chosen fallback if 8443 is busy)
- `postgres` — Mirth's backing database, on a named volume
- `tools` — Alpine + JRE + `mirthsync` (and curl/git/jq) for running CLI operations inside the stack

![Local Mirth](https://raw.githubusercontent.com/SagaHealthcareIT/mirthsync-vscode/refs/heads/main/media/local-mirth.gif)

Set `mirthsync.localMirth.mirthImageTag` to switch to a different `sagait/engine` tag, or edit `compose.yml` to point at `openintegrationengine/engine` for vanilla OIE.

Then run **`MirthSync: Start Local Mirth`**. The extension auto-creates a *Local Mirth* connection and routes its `mirthsync` calls into the `tools` container, so you don't need mirthsync on your host for this workflow. If 8443 is already in use on your machine, the extension will detect the conflict, suggest an alternate port, and remember the choice in `.mirthsync/local/.env` for next time.

To open the Mirth Administrator against the local server, install a launcher on your host and point it at the URL shown in `Show Local Mirth Info`:

- **[Ballista](https://github.com/kayyagari/ballista/releases)** — community launcher used with Open Integration Engine. Requires a host JRE 8+.
- **[NextGen MCAL](https://github.com/nextgenhealthcare/connect/releases)** — legacy NextGen Mirth Connect Administrator Launcher. Bundles its own JRE.

Default credentials are `admin` / `admin`; the Administrator will prompt you to change the password on first login.

**Full guide:** [How to install Mirth Connect, OIE, or BridgeLink with Docker (2026 quickstart)](https://saga-it.com/blog/mirth-connect-quickstart) · [Local Mirth docs](https://saga-it.com/docs/mirthsync/vscode/local-mirth/)

## Requirements

- VS Code 1.85.0 or higher
- [mirthsync](https://github.com/SagaHealthcareIT/mirthsync) CLI tool (for sync operations against non-local Mirth servers)
- Mirth Connect 4.5.2+ or [Open Integration Engine (OIE)](https://github.com/OpenIntegrationEngine/engine)
- Docker (only required for the Local Mirth feature)

### Compatibility

This extension works with:
- **Mirth Connect** - The commercial integration engine by NextGen Healthcare
- **Open Integration Engine (OIE)** - The open source community edition (formerly Mirth Connect Open Source)

Both use the same underlying API, so MirthSync works seamlessly with either.

### Installing mirthsync

The preferred method is via npm:

```bash
npm install -g @saga-it/mirthsync
```

Alternatively, download from [GitHub releases](https://github.com/SagaHealthcareIT/mirthsync/releases).

## Telemetry

This extension sends a small amount of anonymous usage data — command
IDs and durations, success/failure outcomes, numeric error codes,
non-default-setting booleans, and VS Code's standard environment
identifiers. We **never** send channel/template/script content, server
URLs, usernames, credentials, file paths, or subprocess output.

**Opt out** with either:

- `telemetry.telemetryLevel` = `off` (VS Code-wide), or
- `mirthsync.telemetry.enabled` = `false` (this extension only).

Data lands in Azure Application Insights (Saga IT, LLC), 90-day
retention. See [TELEMETRY.md](./TELEMETRY.md) for the full collected /
never-collected lists and how to audit events live via the **MirthSync
Telemetry** output channel.

## Related Projects

- [mirthsync](https://github.com/SagaHealthcareIT/mirthsync) - CLI tool for syncing Mirth configurations
- [Open Integration Engine (OIE)](https://github.com/OpenIntegrationEngine/engine) - Open source integration engine

## About

This is a community extension developed by [Saga IT, LLC](https://saga-it.com). It is **not affiliated with or endorsed by NextGen Healthcare**.

## License

Copyright (c) 2024-2026 Saga IT, LLC. All Rights Reserved.

This is proprietary software. Use is governed by the license shipped
inside the extension package; contact [saga-it.com](https://saga-it.com)
for licensing inquiries.
