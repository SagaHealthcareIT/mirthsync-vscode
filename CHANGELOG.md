# Changelog

All notable changes to the MirthSync VS Code extension are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions with an odd minor number (e.g. `0.1.x`) are published to the Marketplace as pre-releases; even minor versions are stable.

## [Unreleased]

## [0.4.4] - 2026-06-09

Stable patch. Makes anonymous usage telemetry actually work in published
builds — it was silently disabled in all prior releases — and adds
distribution-channel and host-editor attribution. Also trims what ships
in the package.

### Fixed

- **Telemetry delivery from published builds.** Production minification
  was mangling internal property names the bundled telemetry SDK relies
  on, which silently dropped every event before it left the process; and
  the Azure connection string was not being baked into the packaged
  build. Both are corrected, so opted-in anonymous usage telemetry now
  transmits from Marketplace and Open VSX installs. Telemetry still only
  runs when both VS Code's telemetry and the extension's own setting are
  enabled — see [TELEMETRY.md](TELEMETRY.md).

### Added

- **Channel and editor attribution.** Each telemetry event now records
  the distribution channel it was installed from (`marketplace` /
  `openvsx`) and the host editor (`vscode` / `cursor` / `vscodium` / …),
  so usage can be understood per registry and per application.

### Changed

- **Smaller, cleaner package.** The published `.vsix` no longer includes
  internal infrastructure or CI files; it ships only the runtime bundle
  and the assets the extension actually needs.

## [0.4.3] - 2026-06-08

Stable patch. Documentation only — no code or behavior changes.
Corrects the README's Mirth Connect compatibility description, which
backwards-labeled Mirth Connect as the commercial NextGen product.
This is the README baked into the Marketplace and Open VSX listings.

### Changed

- **Compatibility section** now describes the open-source lineage the
  extension actually targets — open-source Mirth® Connect (through
  4.5.2, the last open-source release), Open Integration Engine (OIE),
  and BridgeLink — and clarifies it does **not** target the
  closed-source commercial version (NextGen Connect 4.6+).
- **Requirements** wording corrected from "Mirth Connect 4.5.2+" to
  name the open-source lineage explicitly.
- Added a Mirth trademark note to the non-affiliation disclaimer and
  contextual links to saga-it.com resources.

## [0.4.2] - 2026-05-22

Stable patch. Changes the default Local Mirth image from
`sagait/engine:4.5.2-alpine-jre` (musl libc) to
`sagait/engine:4.5.2-ubuntu-jre` (glibc). Same Mirth version, same
plugins, same `oiecommand`; just a glibc base, which matches what
Mirth's official Linux installer targets and avoids known musl edge
cases around JNI / native libraries used by JDBC drivers, TLS, and
ICU.

### Changed

- **Default `mirthsync.localMirth.mirthImageTag`** is now
  `4.5.2-ubuntu-jre` (was `4.5.2-alpine-jre`). The scaffolded
  `compose.yml` template, settings-snapshot telemetry default, and
  scaffold README are updated accordingly.

### Migration impact (existing Local Mirth workspaces)

On the first **MirthSync: Start Local Mirth** after upgrading, users
who never customized `mirthsync.localMirth.mirthImageTag` will see:

1. A ~340 MB image pull (`sagait/engine:4.5.2-ubuntu-jre`).
2. The `mirthsync-local-mirth` container is recreated against the
   new image.
3. **Data is preserved.** Mirth appdata, custom extensions, and
   Postgres data live in named volumes (`mirth-appdata`,
   `mirth-custom-extensions`, `postgres-data`) and are not touched.
   Channels, code templates, and DB content carry over.

Users who explicitly pinned `mirthsync.localMirth.mirthImageTag`
keep their pinned value with no change.

## [0.4.1] - 2026-05-21

Stable patch. Documentation-only: trims the Marketplace listing's
**Telemetry** section to a short summary and moves the full
collected / never-collected lists and inspect-the-output-channel
walkthrough into a new `TELEMETRY.md` linked from the README. No
runtime changes — what we collect and how to opt out are unchanged.

### Changed

- **README Telemetry section** shortened from ~70 lines to a ~12-line
  summary plus a link to `TELEMETRY.md`. The full content is now in
  `TELEMETRY.md`, mirrored into the companion repo by the release
  workflow alongside `README.md`, `CHANGELOG.md`, and `LICENSE`.

## [0.4.0] - 2026-05-20

Stable release. No functional changes from `0.3.1`; promoted to the
Marketplace stable channel after field-test.

## [0.3.1] - 2026-05-20

Pre-release. Fixes the 0.3.0 Marketplace listing so it renders against
a dedicated public companion repo instead of the unrelated mirthsync
CLI repo, wires release-time mirroring of user-facing files into that
companion repo, and adds opt-in orphan deletion for `Pull All`.

### Added

- **Orphan deletion on `Pull All`** (opt-in). New settings
  `mirthsync.deleteOrphaned` (boolean, default `false`) and
  `mirthsync.promptForDeleteOrphaned` (boolean, default `true`) wire the
  underlying mirthsync CLI's `--delete-orphaned` flag through to the
  extension. When the setting is on and the prompt is on (default), a
  modal at the start of `Pull All` asks whether to proceed with delete,
  proceed without delete, or cancel. When the prompt is suppressed,
  orphans are deleted silently — for unattended workflows. Scoped
  pulls (single channel / channel group / code template library) are
  intentionally unaffected: orphan detection requires a global view of
  what exists on the server. Both settings are in
  `restrictedConfigurations` so a workspace's `.vscode/settings.json`
  cannot silently enable the destructive path. The
  `mirthsync.pullComplete` telemetry event grows a `deleteOrphaned`
  boolean (always set on pull events — `true` only when the user opted
  in via setting + confirmation; scoped pulls always emit `false`) so
  we can see opt-in rates.

### Fixed

- Marketplace listing now resolves correctly against a dedicated public
  companion repo (`SagaHealthcareIT/mirthsync-vscode`) rather than the
  unrelated public `mirthsync` CLI repo. The 0.3.0 listing's relative
  links (`./infra/telemetry/`, `./doc/PLAN-TELEMETRY.md`, `./LICENSE`)
  resolved to the CLI repo and either 404'd or rendered the wrong
  (Eclipse-licensed) `LICENSE`; the README screen-recording GIFs lived
  under that CLI repo too. The README now inlines the proprietary
  license clause directly, `repository` points at the companion repo so
  Marketplace's sidebar link is meaningful, `bugs.url` points at the
  companion's `/issues` so "Report a problem" reaches a real triage
  queue, and the README's GIFs are served from the companion repo's
  `media/`.

### Infrastructure

- Release workflow mirrors `README.md`, `CHANGELOG.md`, and `LICENSE`
  into the public companion repo on tag push, so the Marketplace
  listing's resources stay in sync without manual copy-paste between
  the private source repo and the public-facing one. Authenticates
  with a dedicated ed25519 deploy key on the companion repo
  (`COMPANION_DEPLOY_KEY` secret) — scoped, no expiry, no user
  attribution.

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
