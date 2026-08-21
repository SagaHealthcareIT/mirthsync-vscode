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
  success or failure, duration, and which runtime ran the command —
  the mirthsync installed on your machine (`host`) or the one in the
  Local Mirth tools container (`container`). **Never** the channel,
  template, or script names being operated on.
- Whether the extension found a mirthsync CLI installed on your
  machine — a single boolean, once per session. Never the path where
  it was (or was not) found. This tells us how many people rely on the
  containerised CLI, which decides whether we build a container
  runtime for remote servers.
- Connection counts (bucketed: 1, 2-3, 4-10, 10+) and whether each
  connection points at localhost vs. a remote host. Never the URL.
- Local Mirth lifecycle events (init, start, stop, reset, remove,
  status) and whether Docker is available — a boolean, how long the
  check took, and on failure one of four fixed reason codes
  (`not-found`, `daemon`, `timeout`, `compose`). Never the probe's
  output or error text.
- Error codes from our internal error classification (numeric codes
  only, e.g. `1001`, `4004`, mapped to categories like `Connection`,
  `API`). For certificate errors only, one extra boolean: whether the
  connection was set to Development SSL mode. A certificate error on a
  Development connection means the extension is not applying that
  setting, and without the boolean that bug is indistinguishable from
  the ordinary "this server uses a self-signed certificate" case.
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
- Whether the one OpenShare offer was shown, which surface showed it
  (the post-use notification, or the "can't reach this server?" prompt
  after a connection failure), and which button you pressed —
  including *Don't show again*. Whether you were a fresh install or an
  upgrade. Never anything about the connection that failed.
- Whether an OpenShare sign-in or account creation completed, how many
  servers your account can reach (bucketed: 0, 1, 2-3, 4-10, 10+),
  whether a tunnel was established and which transport path it took,
  and whether a pull/push ran over a tunnel. **Never** your OpenShare
  account, email, organisation, server names or ids, or the console
  address.

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
