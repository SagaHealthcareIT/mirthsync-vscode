# MirthSync Telemetry — full details

The extension README has a short summary. This document is the full,
exhaustive list for anyone who wants to know exactly what the
extension sends, what it never sends, and how to verify it for
themselves.

## Why we collect anything

The data shapes our roadmap. Without it we are guessing at which
commands matter, which Local Mirth lifecycle steps trip people up, and
which can be retired. Every event is anonymous and the property bag is
small.

## What we collect

- Which commands you run (the command ID — e.g. `mirthsync.pull` — and
  how long it took to complete).
- High-level outcomes of MirthSync pull/push and git operations:
  success or failure, duration. **Never** the channel, template, or
  script names being operated on.
- Connection counts (bucketed: 1, 2-3, 4-10, 10+) and whether each
  connection points at localhost vs. a remote host. Never the URL.
- Local Mirth lifecycle events (init, start, stop, reset, remove,
  status) and a boolean for whether Docker is available.
- Error codes from our internal error classification (numeric codes
  only, e.g. `1001`, `4004`, mapped to categories like `Connection`,
  `API`).
- Which non-default settings you have enabled — as booleans. For path
  / URL / image-tag settings, only the boolean "has the user
  overridden the default?" is sent, never the value.
- VS Code's automatically-attached identifiers: OS, OS version, VS
  Code version, extension version, anonymised machine ID, session ID,
  UI kind (desktop / web), remote name (none / WSL / SSH /
  dev-container).
- The distribution channel the build came from — `marketplace` or
  `openvsx` — baked into the build so we can tell which registry our
  installs come from. This is build metadata, not anything about you.
- The host editor's URI scheme — e.g. `vscode`, `cursor`, `vscodium`,
  `code-oss` — i.e. which app you run MirthSync in. A short, fixed
  identifier; never anything user-specific.

## What we will never collect

- **Channel names or content** (transformer scripts, filter scripts,
  connector configuration, deployment scripts).
- **Code template names or content.**
- **Global script content.**
- **ConfigurationMap content.**
- **Server URLs, hostnames, or ports.** We derive a boolean
  `isLocalhost` from the host; the URL itself never leaves your
  machine.
- **Usernames** (Mirth or OS).
- **Passwords, tokens, keys, certificates.**
- **File paths** of any kind. Workspace paths, mirthsync paths, custom
  keystore paths, none of these.
- **Workspace folder names** or git remote URLs.
- **stdout / stderr** from any subprocess (`mirthsync`,
  `docker compose`, `oiecommand`).
- **Free-form text** you type into any input box (connection names,
  commit messages, etc.).
- **Anything from the workspace filesystem** beyond the booleans
  listed above.

## How to opt out

Two switches; either one is enough to stop all sending:

- **VS Code-wide:** set `telemetry.telemetryLevel` to `off`. This
  affects every extension, not just this one. Recommended if you
  don't want any extension reporting telemetry.
- **Just this extension:** set `mirthsync.telemetry.enabled` to
  `false`.

When telemetry is off, the extension logs a one-line confirmation to
the **MirthSync Telemetry** output channel on activation so you can
verify it took effect.

## How to inspect what's being sent

Set `mirthsync.telemetry.showOutput` to `true` (or run **MirthSync:
Show Telemetry Output**) and every event we send — including the full
property bag — is logged to the **MirthSync Telemetry** output channel
as it goes out. You can flip this on at any time to audit exactly
what's being collected.

## Where the data goes

Azure Application Insights, in a resource group owned by Saga IT,
LLC. Retention: 90 days.
