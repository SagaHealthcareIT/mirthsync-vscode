# Changelog

All notable changes to the MirthSync VS Code extension are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions with an odd minor number (e.g. `0.1.x`) are published to the Marketplace as pre-releases; even minor versions are stable.

## [0.6.6] - 2026-11-06

Stable patch release for mirthsync 3.9.0, which gives some files new names. Update the extension when you update mirthsync; it also works with older mirthsync versions.

### Changed

- **mirthsync 3.9.0 is the expected version** (was 3.8.1), and Local Mirth's tools image now installs it. A host mirthsync older than 3.9.0 gets the **Update via npm** prompt; it keeps working, with the file names it writes. Existing Local Mirth workspaces move to 3.9.0 the next time Local Mirth starts, and if any item names change, the notice below appears once before the first push or pull.
- **Ready for mirthsync 3.9.0.** mirthsync 3.9.0 gives files new names when an item's name has characters Windows doesn't allow in file names (`:` becomes `%3A`, `%` becomes `%25`) or is a Windows device name such as `AUX`. Push and pull from the Channels and Code Templates views now build the item's path the way the mirthsync you run names it, so they work with 3.9.0 and with older versions.
- **One notice before the first mirthsync 3.9.0 push or pull** in a workspace that has names 3.9.0 writes differently, with the steps from mirthsync's upgrade notes: push any changes made with the older version, then pull everything once with both Force Sync and Delete Orphaned on (`mirthsync pull -f --delete-orphaned`) and commit. A first 3.9.0 pull with Delete Orphaned on but Force Sync off is stopped, because it would delete script files that channels still use.
- Push and pull messages show the item's own name, not its encoded file name.
- **The MirthSync output shows which mirthsync each push or pull runs**, and where it comes from (the installed mirthsync or the Local Mirth tools container). It also notes when the installed mirthsync changed version.

### Fixed

- **A push or pull that matched nothing no longer says it succeeded.** With mirthsync 3.9.0 or later, the extension warns that nothing was pushed or pulled.
- Updating mirthsync while VS Code is open is picked up on the next push or pull, without a reload.
- Saving before a push now includes the channel's or template's own `.xml` file, not only the scripts in its folder.

## [0.6.5] - 2026-11-05

Stable patch release with security hardening. Recommended for everyone.

### Security

- **mirthsync and the Mirth CLI are now started directly, not through a command shell.** Names that come from a connected engine (channel, group, code template and library names) are always passed to them as plain text. Update to this release.
- **Your mirthsync password is no longer on the command line.** The extension passes it in the `MIRTHSYNC_PASSWORD` environment variable, which mirthsync reads. Local Mirth connections are unchanged.

### Changed

- **On Windows with mirthsync older than 3.6.0** (check with `mirthsync --version`), an item whose name contains a character such as `&`, `|`, `%` or `"` is not pulled or pushed. The extension asks you to update mirthsync (`npm install -g @saga-it/mirthsync`) instead.
- **A custom `.cmd` or `.bat` launcher** set in `mirthsync.mirthsyncPath` still works, but an operation whose values contain `"`, `%` or a line break is refused with a message. The standard npm install and the mirthsync release download are not affected.

### Fixed

- Passwords containing `$`, a backtick or `\` now work for mirthsync pull and push.
- A mirthsync install path with spaces (for example under `C:\Program Files\nodejs`) no longer breaks detection.
- The Mirth CLI deploy, undeploy, import and export commands pass channel names and file paths with spaces exactly as typed.

## [0.6.4] - 2026-11-02

Stable patch release. Local Mirth moves to Open Integration Engine 4.7.0 and mirthsync 3.8.1.

### Changed

- **Default `mirthsync.localMirth.mirthImageTag`** is now
  `4.7.0-ubuntu-jre` (was `4.6.0-ubuntu-jre`). `sagait/engine` was
  first published on OIE 4.7.0 on 2026-10-30, bundling mirthsync-plugin
  0.0.6, tls-manager 1.0.9 and simple-channel-history 2.9.0. The scaffolded
  `compose.yml`, its README and the settings-snapshot telemetry default
  are updated to match.
- **A new *Local Mirth* connection gets a generated password.** OIE 4.7
  has no built-in `admin` / `admin`: when it creates its database it sets
  the admin password from `_MP_SERVER_INITIALADMINPASSWORD`, or generates
  one and prints it once to the engine log. When the extension creates the
  *Local Mirth* connection it now saves a random 24-character password
  (letters and digits) instead of `admin`, and **Start Local Mirth** passes
  the connection's saved password to the engine in that variable. The
  managed `docker-compose.override.yml`, now written on every Start, names
  the variable without a value, so the password comes from Start's
  environment and is not written to any file in the workspace. The Start
  message and **Show Local Mirth Info** no longer say `admin` / `admin`;
  both copy the saved password.
- **Bundled mirthsync pinned to 3.8.1** (was 3.7.0). The Local Mirth tools
  image (`mirthsync.localMirth.mirthsyncVersion`), the expected-version
  check and the outdated-CLI prompt now target mirthsync 3.8.1, so a host
  CLI older than 3.8.1 now gets the **Update via npm** prompt. Selective
  deployment still needs only 3.7.0. From the mirthsync release notes:
  - 3.8.0 sorts the collections Mirth stores in no fixed order (code
    template context types, channel dependency and tag IDs, the
    configuration map and others) when it pulls, so pulling unchanged code
    no longer reorders them in version control. Expect a one-time commit
    that only reorders these elements the first time you pull over existing
    files with `mirthsync.forceSync` on.
  - 3.8.1 makes orphan detection on pull linear instead of quadratic. On a
    server with about 300 channels, a pull dropped from about 9 minutes to
    about 20 seconds. The old check also missed some stale `.js` files, so
    **Pull All** with `mirthsync.deleteOrphaned` on may now report, and
    delete, script files it skipped before.

### Migration impact (existing Local Mirth workspaces)

**Initialize Local Mirth** copies the template into `.mirthsync/local/`,
and later extension updates don't rewrite those files. **Start Local
Mirth**, though, passes the `mirthImageTag` and `mirthsyncVersion` settings
to `docker compose`, and they take priority over the versions in the
workspace's own `compose.yml` and over any `MIRTH_IMAGE_TAG` or
`MIRTHSYNC_VERSION` line in `.mirthsync/local/.env`. So on the first Start
after upgrading, users who never customized those settings will see:

1. A pull of `sagait/engine:4.7.0-ubuntu-jre`, and the
   `mirthsync-local-mirth` container recreated on it.
2. The tools image rebuilt with mirthsync 3.8.1.
3. **A one-way database upgrade.** Channels, code templates and the rest of
   the database carry over, but 4.7.0 migrates the schema on its first
   start and a 4.6.0 engine fails to start against it afterwards. To stay
   on 4.6.0, set `mirthsync.localMirth.mirthImageTag` to
   `4.6.0-ubuntu-jre` *before* that first Start. If the database has
   already been upgraded, set the tag to `4.6.0-ubuntu-jre` and then run
   **MirthSync: Reset Local Mirth** to wipe it and start fresh.
4. **Your admin password is kept.** The database already has it, so
   `admin` / `admin`, or whatever you changed it to, still works.
5. **Nothing to re-initialize.** The *Local Mirth* connection keeps its
   saved password (usually `admin`), and Start seeds every new database
   with it: after **Reset Local Mirth** or a manual `docker compose down
   -v`, on the first Start of a workspace that was initialized but never
   started, and in a fresh clone of a repository that commits
   `.mirthsync/local/compose.yml`. The seed goes through the managed
   `docker-compose.override.yml`, so the workspace's existing `compose.yml`
   needs no change. If you wrote your own `docker-compose.override.yml`,
   which the extension never edits, list `_MP_SERVER_INITIALADMINPASSWORD`
   with no value under `services.mirth.environment`; Start warns while it
   is missing.

The saved password and the database can still disagree: when you change the
password in the Administrator, when the *Local Mirth* connection is deleted
while its database survives (the recreated connection gets a new generated
password), or when an engine older than 4.7, which ignores the seed and
always creates `admin` / `admin`, creates the database for a new connection.
Connecting, pulling or pushing then reports that the server rejected the
stored credentials and offers **Update Credentials**. If you don't know the
database's password, **Reset Local Mirth** and Start again: on 4.7 and later
the new database gets the saved password.

Users who pinned `mirthsync.localMirth.mirthImageTag` or
`mirthsync.localMirth.mirthsyncVersion` in their VS Code settings keep
those values.

## [0.6.3] - 2026-09-24

Stable patch release. Push now offers to save unsaved files first.

### Added

- **Save before push** ([mirthsync#94](https://github.com/SagaHealthcareIT/mirthsync/issues/94)).
  mirthsync pushes files from disk, so unsaved edits were silently left
  out. Push now lists unsaved files in the push's scope (the whole
  workspace, or the selected channel, group, or library) and offers
  **Save All & Push**, **Push Without Saving**, or cancel. New setting
  `mirthsync.saveBeforePush`: `prompt` (default), `always`, or `never`.
  If a save fails, the push is cancelled.

## [0.6.2] - 2026-09-15

Stable release for mirthsync 3.7.0. Selective deployment after push, and
Initialize Local Mirth now pulls the engine image so a cached, stale tag
can no longer start a build whose bundled plugins the engine rejects.

### Added

- **Selective deployment after push** (mirthsync 3.7.0). New setting
  `mirthsync.deployStrategy` chooses how `deployAfterPush` deploys: `all`
  (every pushed channel, `--deploy`, the previous behaviour and still the
  default), `changed` (only channels the server reports as changed after
  the push, `--deploy-changed`), or `changed-and-new` (changed channels
  plus pushed channels that are not currently deployed,
  `--deploy-changed --deploy-new`). **MirthSync: Select Deploy Strategy**
  offers the same choice as a quick pick. The flags are only passed when
  the mirthsync that will run is 3.7.0 or newer: the detected host CLI for
  normal connections, the `mirthsync.localMirth.mirthsyncVersion` tools
  image for Local Mirth. On an older CLI the push falls back to `--deploy`
  and says so once.

### Changed

- **Bundled mirthsync pinned to 3.7.0.** The Local Mirth tools image, the
  expected-version check, and the outdated-CLI prompt now target
  mirthsync 3.7.0. Existing Local Mirth workspaces rebuild the tools image
  on the next Start; until then a tools container that is still running
  from before the update has 3.6.0, so run **Start Local Mirth** once
  before choosing a selective deploy strategy against it.

- **Initialize Local Mirth now pulls the engine image.** `docker compose up`
  never re-downloads a tag that already exists locally, so a machine that had
  cached `sagait/engine:<tag>` before the tag was rebuilt kept starting the
  stale build, with older bundled plugins, and the only sign was an
  "Extension ... is not compatible" line in the engine log. Initialize now
  runs `docker compose pull mirth` for the configured
  `mirthsync.localMirth.mirthImageTag` right after scaffolding. The pull is
  cancellable; a failed or cancelled pull is reported and Start proceeds with
  whatever is cached. Docker being unavailable at Initialize is not an error,
  as before.

## [0.6.1] - 2026-09-15

Stable patch. Moves the default Local Mirth image from
`sagait/engine:4.5.2-ubuntu-jre` to `sagait/engine:4.6.0-ubuntu-jre`,
the current Open Integration Engine release. The `4.5.2` tags have been
frozen since May and no longer receive plugin or base-image updates;
`4.6.0-ubuntu-jre` was rebuilt on 2026-09-01 with mirthsync-plugin
0.0.4, simple-channel-history 2.8.0, and tls-manager 1.0.7.

### Changed

- **Default `mirthsync.localMirth.mirthImageTag`** is now
  `4.6.0-ubuntu-jre` (was `4.5.2-ubuntu-jre`). The scaffolded
  `compose.yml` template, settings-snapshot telemetry default, and
  scaffold README are updated accordingly.
- The scaffold README no longer claims `admin-console` is bundled in
  `sagait/engine`; the image ships mirthsync, simple-channel-history,
  and tls-manager. The OpenShare gateway plugin is installed separately
  through the `mirth-custom-extensions` volume.

### Migration impact (existing Local Mirth workspaces)

On the first **MirthSync: Start Local Mirth** after upgrading, users
who never customized `mirthsync.localMirth.mirthImageTag` will see:

1. A ~370 MB image pull (`sagait/engine:4.6.0-ubuntu-jre`).
2. The `mirthsync-local-mirth` container is recreated against the
   new image.
3. **Data is preserved.** Mirth appdata, custom extensions, and
   Postgres data live in named volumes (`mirth-appdata`,
   `mirth-custom-extensions`, `postgres-data`) and are not touched.
   4.6.0 makes no schema changes; channels, code templates, and DB
   content carry over as they are.
4. **Going back is not supported.** On first start 4.6.0 stamps the
   database with its own schema version, which a 4.5.2 engine does not
   recognise and will not start against. To stay on 4.5.2, set
   `mirthsync.localMirth.mirthImageTag` to `4.5.2-ubuntu-jre` *before*
   the first start on this version, or run **MirthSync: Reset Local
   Mirth** afterwards to wipe the volumes and start fresh.

Users who explicitly pinned `mirthsync.localMirth.mirthImageTag`
keep their pinned value with no change.

## [0.6.0] - 2026-08-20

A stable release rather than the usual pre-release first: nobody
meaningfully consumes the pre-release channel, and the fix for
[#81](https://github.com/SagaHealthcareIT/mirthsync/issues/81) is needed
by people on stable today.

The OpenShare features go live in this build. `api.openshare.health`
now serves the client configuration the extension bootstraps from, so
sign-in and tunnels work against production for the first time.

### Added

- **A Get Started page, which opens itself once.** It covers the whole
  arc: add a connection, pull and push channels, reach a remote engine
  with OpenShare, and register an engine in the OpenShare Console. It
  opens on its own the first time you run this version, whether you
  installed fresh or updated into it, and never again after that. If you
  have turned off VS Code's
  `workbench.welcomePage.walkthroughs.openOnInstall`, it stays shut.
  Finished steps check themselves off (on a sync that actually succeeded,
  not on one you started), and the OpenShare half appears only when
  OpenShare is reachable, changing as you sign in and register a server.
  Reach it any time with **MirthSync: Get Started**.
- **An illustrated Get Started walkthrough** is also contributed, so
  MirthSync appears on VS Code's own Welcome page. Note that VS Code
  hides walkthrough artwork when the editor is 950px or narrower, which a
  split editor usually is; the Get Started page above is laid out to work
  at any width, which is why it is the one that opens.
- **A cloud button in the Mirth Connections view title** for **List
  OpenShare Servers**, which until now was reachable only from the command
  palette.
- **Create an OpenShare account without leaving VS Code.** The new
  **MirthSync: Create an OpenShare Account** command runs the same sign-in
  flow against the identity provider's registration form, so you register,
  verify your email, and land back in VS Code signed in.
- **MirthSync: Register a Server in the OpenShare Console**, which opens
  the console's Servers page. Offered automatically when a listing comes
  back empty, alongside the walkthrough step explaining the sequence.
- **One optional notification about OpenShare**, and only one: it appears
  after you have actually used the extension (a completed pull, push, or a
  created connection), never on a bare start-up, and it can be silenced
  permanently from the notification itself. If a connection fails in the
  specific way a tunnel fixes (refused, timed out, or DNS failure against a
  remote host), that offer arrives there instead. The two share a single
  budget, so you see at most one of them, once. Neither appears if
  OpenShare cannot be reached, or if you are already signed in. Set
  `mirthsync.openshare.promotions` to `false` to suppress the notification
  and the welcome-view block entirely; the OpenShare commands and the
  walkthrough stay available either way.

### Fixed

- **Signing up no longer runs out of time while you are still signing
  up.** MirthSync waits for your browser to come back after sign-in, and
  the wait was five minutes for everyone. Creating an account takes
  longer than that more often than not, between one-time codes, email
  verification, and the terms, and it failed right at the end after all
  the work was done. Registration now gets twenty minutes and ordinary
  sign-in ten. If it does run out, nothing is lost: run the command
  again.

- **The Get Started page now actually opens after an upgrade.** It is
  meant to open once when you first run a new version, but the
  extension only started up at all when your workspace already looked
  like a Mirth project or you clicked the Mirth Connections view, so
  for most people nothing happened. MirthSync now starts with the
  editor. Whether OpenShare is reachable is answered from a local cache
  that lasts a week, so in normal use this costs one small request a
  week and nothing at all in between; if OpenShare cannot be reached,
  MirthSync waits a day before trying again rather than retrying every
  time you open a window.

- **Development connections now actually accept self-signed
  certificates.** Setting a connection's SSL mode to Development had no
  effect on the extension's own requests (Connect, Test Connection,
  refreshing the tree) to any server that was not on your own machine:
  they kept failing with `[2001] unable to verify the first
  certificate`. Pull and push were unaffected, because those run the
  mirthsync CLI, which got the setting correctly. VS Code replaces the
  certificate settings we attach to a connection whenever its
  `http.proxySupport` setting is left at its default, which it is for
  almost everyone, and whether or not you use a proxy at all. Local
  connections are exempt from that, which is why Local Mirth always
  worked and why this went unnoticed for so long. The settings are now
  applied in a way VS Code preserves. Proxies still work, and Production
  connections still refuse bad certificates. If you worked around this
  with `"http.proxySupport": "off"`, you no longer need it and can
  remove it. (SagaHealthcareIT/mirthsync#81)
- **A certificate error on a Development connection no longer repeats
  advice you already followed.** It used to tell you to switch the
  connection to Development when it was already set to Development. It
  now says the certificate was rejected anyway and points at the likely
  causes.

- **A brand-new OpenShare account no longer looks like a broken sign-in.**
  Until the terms are accepted in the console, the API answers 403 to a
  perfectly valid session, and the extension reported that as "not
  authorized, sign out and back in" — advice that cannot possibly clear it.
  It now says what is actually needed and offers to open the console.

- **Docker detection now tells you what is actually wrong.** The
  availability check previously collapsed every failure into one "Docker is
  not available" message. It now distinguishes the CLI missing from VS
  Code's PATH (with a hint about launching VS Code from a terminal — on
  macOS, Docker Desktop installs the CLI to `~/.docker/bin`, which
  Finder-launched apps don't see), a daemon that is installed but not
  responding, a probe that timed out (new 10-second cap; previously a
  wedged daemon could hang the command indefinitely), and a missing
  Compose v2 plugin. The error notification gains a **Retry** button, since
  the common recovery is simply "start Docker Desktop and try again."
- **Removing Local Mirth no longer claims success when Docker cleanup was
  skipped.** When Docker is unavailable (or `docker compose down` fails),
  the final message now says the containers, volumes, and tools image may
  remain and gives the `docker compose -p mirthsync-local down -v --rmi
  local` command to finish the job once Docker is back.

### Changed

- The anonymous `localMirth.dockerAvailability` telemetry event now
  includes the check duration and, on failure, which of the four fixed
  reasons applied (`not-found`, `daemon`, `timeout`, `compose`) — never
  the probe's output. See TELEMETRY.md.
- Two anonymous signals now record **which mirthsync ran your sync**: the
  pull / push / git events carry `runtime` (`host` for the CLI installed
  on your machine, `container` for the one in the Local Mirth tools
  service), and a new once-per-session `mirthsync.cliDetection` event
  carries a single `detected` boolean — whether the extension found a
  host-installed CLI. Neither sends a path. This decides whether we build
  a containerised CLI runtime for remote servers, which today is not
  possible: the container route only reaches the bundled Local Mirth.
  See TELEMETRY.md.

### Removed

- **Scaffolded `.devcontainer/devcontainer.json`** from the Local Mirth
  template. When the root `.devcontainer/` stack was removed, this one was
  kept as "a real feature for Codespaces / Remote-SSH users of Local Mirth"
  — that reasoning was half wrong, and telemetry says the audience is one
  machine. Under Remote-SSH the extension host already runs on the remote
  with its own Docker, so those users never needed this file. Codespaces
  (or a local "Reopen in Container") is the case it existed for, and there
  it lands the extension host inside the `tools` container, where mirthsync
  invocation spawns `docker compose exec` against a container that has no
  Docker CLI and no socket — nothing works. The one session that ever used
  it failed end to end. Workspaces that already scaffolded the file are
  left untouched; it is inert unless deliberately opened in a container.

## [0.4.5] - 2026-06-12

Stable patch. Updates the extension for mirthsync 3.6.0, fixes argument
quoting when the bundled CLIs are spawned, and adds a heads-up when the
installed mirthsync is older than the version this release expects.

### Added

- **mirthsync version detection and outdated notice.** The extension now
  reads the installed mirthsync version via `mirthsync --version` and shows
  a non-blocking notification — with *Update* and *Release Notes* actions —
  when it is older than the expected 3.6.0.

### Fixed

- **Argument quoting for spawned CLIs.** Arguments containing spaces or
  shell metacharacters — passwords, tokens, server URLs, channel names, the
  git email, and file paths — are now quoted when mirthsync and mirth-cli
  are invoked through a shell, preventing word-splitting that could break
  commands (most visibly on Windows).

### Changed

- **Bundled mirthsync pinned to 3.6.0.** The Local Mirth Docker tools image
  and the expected-version check now target mirthsync 3.6.0, which adds the
  `--version` flag and fixes Windows `--restrict-to-path` handling of spaces
  and forward slashes along with the `--delete-orphaned` capture bug.

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
