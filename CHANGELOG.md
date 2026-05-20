# Changelog

All notable changes to the MirthSync VS Code extension are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions with an odd minor number (e.g. `0.1.x`) are published to the Marketplace as pre-releases; even minor versions are stable.

## [Unreleased]

### Fixed

- Marketplace listing no longer resolves to the unrelated public
  `mirthsync` CLI repo. Dropped the relative `./infra/telemetry/`,
  `./doc/PLAN-TELEMETRY.md`, and `./LICENSE` links from the README
  (they 404'd or showed the wrong, Eclipse-licensed file); removed
  the `repository` field from `package.json` (the extension source is
  in a private repo, so a public repo URL would have been misleading);
  pointed `bugs.url` at `https://saga-it.com/products/mirthsync-vscode#support`
  so "Report a problem" reaches the right team instead of the CLI
  maintainers.

## [0.3.0] - 2026-05-14

Pre-release. First pass of Local Mirth, anonymous opt-out telemetry, and
a Windows-specific Cancel reliability fix. Will be promoted to `0.4.0`
stable after field-test.

### Added

- **Local Mirth.** One-command scaffold of a Docker Compose stack
  (`sagait/engine` — Saga-packaged Open Integration Engine — plus Postgres and
  a `tools` container with mirthsync) into `.mirthsync/local/` of the
  workspace. The extension manages start / stop / reset / status / logs and
  surfaces the URL and credentials for pointing a Mirth Administrator launcher
  at the running server.
  - New commands: `MirthSync: Initialize Local Mirth`, `Start Local Mirth`,
    `Stop Local Mirth`, `Reset Local Mirth (Delete State)`,
    `Remove Local Mirth (Delete All)`, `Show Local Mirth Status`,
    `Show Local Mirth Logs`, `Show Local Mirth Info`.
  - `Remove Local Mirth` is the bigger hammer over `Reset`: it deletes the
    Docker containers, named volumes, locally-built tools image, the
    entire `.mirthsync/local/` directory, and the auto-created
    `Local Mirth` connection profile, restoring the workspace to a
    pristine pre-init state. `Reset` continues to wipe only the Docker
    volumes (Postgres + Mirth appdata), leaving the scaffold and
    connection in place for a fresh-DB restart.
  - New settings: `mirthsync.localMirth.mirthImageTag`,
    `mirthsync.localMirth.mirthsyncVersion`,
    `mirthsync.localMirth.additionalPorts`. The `additionalPorts` array is
    automatically materialized into a managed
    `.mirthsync/local/docker-compose.override.yml` on Start (gitignored;
    deleted when the setting is cleared; never overwrites a user-authored
    override).
  - When the local Mirth is the active connection, `mirthsync` runs inside
    the `tools` container via `docker compose exec`, so a host install isn't
    required for local-only workflows.
  - Host-side port conflicts are detected on start: if the desired port is
    already in use, the extension surfaces the conflicting process (where
    possible), proposes a free fallback, and persists the choice in
    `.mirthsync/local/.env` so subsequent starts use the same port.
  - Auto-creates a "Local Mirth" connection on first successful start and
    keeps its port in sync with whatever host port was resolved.
  - Includes an optional `.devcontainer/devcontainer.json` for Codespaces
    and Remote-SSH users that targets the same compose stack.
  - `MirthSync: Show Local Mirth Logs` is now cancellable — wrapped in a
    progress notification with a Cancel button that actually kills the
    `docker compose logs -f` subprocess (previously the only way to stop
    the log stream was to close VS Code).

### Changed

- Marketplace listing copy refreshed: description, categories, and keywords
  rewritten to reflect what the extension actually does today, with high-intent
  Mirth Connect / OIE / channel-sync keywords surfaced so users searching for
  those terms can find us. The `homepage` field now points at
  https://saga-it.com/products/mirthsync-vscode.
- Removed the `Debuggers` category and dropped the `dev containers` claim from
  the description — neither was accurate.

### Fixed

- **Cancelling a Pull or Push on Windows no longer leaves the JVM running.**
  Node's `child.kill('SIGTERM')` on Windows calls `TerminateProcess` on the
  immediate child handle and does not propagate to grandchildren — and
  `mirthsync` runs via a `.bat` wrapper that spawns the JVM as a grandchild.
  A new `killChildTree` helper (`src/core/processControl.ts`) uses
  `taskkill /T /F` on Windows to walk the process tree, preserving the
  existing SIGTERM-then-SIGKILL behavior elsewhere. Wired into both the
  mirthsync and Mirth-CLI executors.
- **Initializing Local Mirth in a workspace that already has a `.mirthsync`
  marker file** now surfaces a clear "rename or delete the marker before
  re-running" message instead of leaking the raw `EEXIST` mkdir error.

### Removed

- **JVM debugging** support. The `mirth-java` debug type, the
  `MirthDebugConfigurationProvider` (which only ever provided launch-config
  presets and a port-check helper — it never implemented JDWP itself), and the
  `PHASE-5-DEBUGGING.md` aspirational plan have all been deleted. Anyone who
  was using the `mirth-java` launch type can switch to the upstream `java`
  attach configuration provided by Microsoft's Java extension.
- **Settings Panel webview.** The `mirthsync.openPanel` command was not
  reachable from any UI gesture and its webview was a half-built placeholder
  pointing at a broken script path. Removed the command, the entire
  `src/mirthsync/webview/` tree (panel, messaging, CSP, UI entry), the
  `media/panel.css` stylesheet, and the webview webpack bundle.
- **Root `.devcontainer/` stack.** Predated Local Mirth and was tied to a
  JVM-debug launch config that referenced the (now-removed) debugging
  support. Local Mirth covers the same "throwaway Mirth for development"
  use case more cleanly and without forcing everyone to reopen the
  workspace inside a container. Also dropped the matching ~166-line
  "Dev Container Security" section in `doc/SECURITY.md` that documented
  `src/devcontainer/setup.ts` — a file that never existed. The scaffolded
  `.devcontainer/devcontainer.json` in `resources/local-mirth-template/`
  stays — it is a real feature for Codespaces / Remote-SSH users of
  Local Mirth and is self-contained.

### Security

- The Local Mirth settings (`mirthImageTag`, `mirthsyncVersion`,
  `additionalPorts`) are now declared `restrictedConfigurations`, so a
  workspace `.vscode/settings.json` cannot redirect the local stack to an
  unexpected image, mirthsync version, or port forward unless the workspace
  is trusted.
- `composeStatus` now invokes `docker compose` via `execFile` rather than a
  shell, eliminating shell-metacharacter parsing of workspace paths.

### Infrastructure

- Add OpenTofu module at `infra/telemetry/` defining the Azure Application
  Insights stack that the (still-unimplemented) telemetry plan targets:
  resource group, Log Analytics workspace, workspace-based AAI instance,
  90-day retention. Includes a hardened bootstrap script for the remote
  state backend (AAD-auth-only, shared-key access disabled, blob
  soft-delete). Not applied yet — design artifact paired with
  `doc/PLAN-TELEMETRY.md`.
- `.gitignore` now covers OpenTofu / Terraform working files (`.terraform/`,
  `*.tfstate*`, `*.tfvars*`) so the state file — which contains the
  AAI connection string in plaintext — can't accidentally be committed.

### Telemetry

- **Anonymous usage telemetry** (opt-out) is now wired up against the
  OpenTofu-defined Azure Application Insights stack. The connection
  string is baked into release builds at compile time via the
  `AZURE_AI_CONNECTION_STRING` GitHub Actions secret (read by webpack's
  `DefinePlugin`); dev / fork builds get an empty string and the
  reporter no-ops.
  - New `src/telemetry/` module: typed `TelemetryEventName` enum, a
    reporter that honours `vscode.env.isTelemetryEnabled` and the new
    `mirthsync.telemetry.enabled` setting, property sanitization
    (clamps every string to 64 chars; runs basename-only reduction on
    `stack` properties), bucketed counts, an `isLocalhost`
    derivation for hosts, and a `registerInstrumentedCommand` wrapper
    that emits `command.invoked` for every user command.
  - Sync, connection, and local-Mirth lifecycle events fire through
    EventBus subscribers in `extension.ts` rather than emit calls
    sprinkled across modules — single point of egress, single review
    surface.
  - Auditable **MirthSync Telemetry** output channel: set
    `mirthsync.telemetry.showOutput` to `true` (or run **MirthSync:
    Show Telemetry Output**) to see exactly what's emitted, including
    suppressed events and the reason they were suppressed.
  - `README.md` gains a top-level Telemetry section with the exact
    event list, the never-collect list, and opt-out instructions.
- New settings (both in `restrictedConfigurations` so a workspace can't
  flip them on the user's behalf):
  - `mirthsync.telemetry.enabled` (boolean, default `true`)
  - `mirthsync.telemetry.showOutput` (boolean, default `false`)
- New command: `MirthSync: Show Telemetry Output`
  (`mirthsync.telemetry.showOutput`).

## [0.2.0] - 2026-05-04

Stable release. No functional changes from `0.1.12`; promoted to the Marketplace stable channel.

## [0.1.12] - 2026-04-24

### Fixed

- SSL certificate errors now guide users to the per-connection **Development** SSL mode instead of pointing at the `mirthsync.ignoreCertificateWarnings` setting. That setting only affects the `mirthsync` CLI subprocess (pull/push) and has no effect on the in-process HTTPS client used by Connect and Test Connection, so the old guidance led users to a toggle that didn't help. ([#81](https://github.com/SagaHealthcareIT/mirthsync/issues/81))
- **Edit Connection** now prompts for SSL mode. Previously, a connection created as "Production" could not have its SSL mode changed without deleting and recreating it — which made the new error guidance unreachable. Pressing Escape at the prompt keeps the current value.

### Changed

- E2E detection tests now exercise the extension's real `detectMirthSync()` logic via a test-only command, rather than asserting a hard-coded `/usr/local/bin/mirthsync` path. They now pass for any supported install location (npm global, nvm, Homebrew, `~/.local/bin`, etc.).

## [0.1.11] - 2026-02-03

### Fixed

- Corrected image URLs in the README to reference the correct default branch.

## [0.1.10] - 2026-02-03

### Changed

- Moved extension media assets to the public `mirthsync` repository so screenshots and icons render on the Marketplace listing.

## [0.1.9] - 2026-02-03

### Fixed

- Updated extension ID references throughout the codebase after the publisher change to `SagaITLLC`, and added an upgrade notice for users coming from older publisher IDs.

## [0.1.8] - 2026-02-03

### Changed

- Changed Marketplace publisher to `SagaITLLC`.
- Switched to VS Code Marketplace-compatible versioning.
