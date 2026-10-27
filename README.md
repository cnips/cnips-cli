<p align="center">
  <h1 align="center">CNIPS CLI</h1>
  <p align="center">
    <strong>The official command-line interface for the CNIPS Integration Platform</strong>
  </p>
  <p align="center">
    <a href="#installation">Installation</a> •
    <a href="#quick-start">Quick Start</a> •
    <a href="#commands">Commands</a> •
    <a href="#examples">Examples</a> •
    <a href="#configuration">Configuration</a>
  </p>
</p>

---

The CNIPS CLI enables local development, testing, and deployment of integration pipelines, components, and functions. Build and run pipelines locally with hot-reload, sync changes with remote workspaces, and publish components to the marketplace — all from your terminal.

### CLI contracts

- **Login required** for network sync and publish: `pull`, `push`, `resolve`, `publish`, `switch`, and `status` / `diff` with `--remote`.
- **Works offline** (no saved token): `init`, `login`, `logout`, `profile`, `discard`, `add`, `rename`, `fmt`, `validate`, `build`, `run`, `trace`, `stash`, local `status` / `diff`, plus `help`, `completion`, and `version`.
- **Exit codes:** Command failures that already printed a themed error (validate, `fmt --dry-run`, push reject, build target errors, failed pipeline `run`, `diff --exit-code`) exit **1**. User cancel on confirm/menus exits **0**.
- **Output conventions:** `✓` success, `⚠` warning, `✗` error, `›` info (dry runs use `›`, never `✓`). Every "Next steps:" hint is printed after a blank line with each command indented under the heading. Running a project command outside a project prints `✗ Not a cnips project (no cnips.yaml in <dir> or any parent)` with `cnips init` next steps.
- **Scaffold / `--type` kinds:** `source`, `destination`, `transformation`, `approval`, `switch`, `decision`, `function` (legacy type `component` is not supported).

## Features

- **Local Pipeline Execution** — Run pipelines locally with native components (JSONata transformations, decisions, switches, loops, approvals)
- **Hot-Reload Development** — `cnips run function <name> --watch` serves the function and rebuilds on source changes
- **Multi-Language Support** — JavaScript/TypeScript, Python, and Go components and functions
- **Git-like Workspace Sync** — Pull, push, diff, status, stash, and resolve with three-way tracking against a local base
- **Local-first status** — `cnips status` compares against the tracking base with no network call; pass `--remote` to fetch live cnips, conflicts, and auto-mergeable paths
- **Interactive Conflict Resolution** — On pull/stash conflicts, accept incoming (`--accept-cnips` / `--accept-stash`), keep local, or write markers; bulk-resolve later with `cnips resolve`
- **Terminal UX** — Colored status output, progress spinners, arrow-key menus, and ASCII/`NO_COLOR` fallbacks
- **Editor Helpers** — `cnips init` / `cnips pull` install VS Code YAML schema overrides for `component.yaml`
- **Validation & Formatting** — Lint project files, detect secret leaks, and enforce canonical YAML formatting
- **Marketplace Publishing** — Submit components directly to the CNIPS marketplace

## Installation

### Homebrew (recommended)

```bash
brew install cnips/tap/cnips
```

### Binary download

Prebuilt binaries are published on the public distribution repo:

**[github.com/cnips/cnips-cli/releases](https://github.com/cnips/cnips-cli/releases)**

| Platform | Archive |
|----------|---------|
| macOS (Apple Silicon) | `cnips_vX.Y.Z_darwin_arm64.tar.gz` |
| macOS (Intel) | `cnips_vX.Y.Z_darwin_amd64.tar.gz` |
| Linux (x86_64) | `cnips_vX.Y.Z_linux_amd64.tar.gz` |
| Linux (ARM64) | `cnips_vX.Y.Z_linux_arm64.tar.gz` |
| Linux (ARMv7) | `cnips_vX.Y.Z_linux_armv7.tar.gz` |
| Windows (x86_64) | `cnips_vX.Y.Z_windows_amd64.zip` |
| Windows (ARM64) | `cnips_vX.Y.Z_windows_arm64.zip` |

Download the archive for your platform, extract it, and put `cnips` on your `PATH`.

### Verify Installation

```bash
cnips version
cnips --help
```

### Distribution

| What | Where |
|------|--------|
| Release binaries | [github.com/cnips/cnips-cli](https://github.com/cnips/cnips-cli/releases) |
| Homebrew tap | [github.com/cnips/homebrew-tap](https://github.com/cnips/homebrew-tap) (`brew install cnips/tap/cnips`) |


## Quick Start

### 1. Initialize a New Project

```bash
cnips init my-integrations
cd my-integrations
```

This creates the standard project structure:

```
my-integrations/
├── cnips.yaml              # Project manifest
├── cnips.lock              # Dependency lockfile
├── pipelines/              # Pipeline definitions
├── functions/              # FaaS functions
├── sources/                # Source components (extractors)
├── destinations/           # Destination components
├── transformations/        # Transformation components
├── approvals/              # Approval components
├── switches/               # Switch components
├── decisions/              # Decision components
├── environments/           # Environment configurations
│   └── dev.yaml
└── .vscode/                # YAML schema for component.yaml
```

Runtime sync state lives outside the repo in the user config directory (not under the project tree).

### 2. Authenticate with CNIPS

```bash
# Browser-based OIDC login
cnips login --base-url https://your-cnips-instance.com

# Or with a token
cnips login --token "$CNIPS_TOKEN" --workspace default
```

### 3. Pull Existing Workspace

```bash
cnips pull
```

### 4. Create a Component

```bash
# Create a transformation in JavaScript
cnips add normalize-order --type transformation --language javascript

# Type-first form (same result)
cnips add transformation normalize-order --language javascript

# Create a Go function (always Fiber for Go functions)
cnips add webhook-handler --type function --language go

# Create a Python source extractor
cnips add api-fetcher --type source --language python
```

### 5. Run Locally

```bash
# Run a pipeline with JSON payload
cnips run my-pipeline --payload '{"orderId": "12345"}'

# Run with payload from file
cnips run my-pipeline --payload-file ./test-data/order.json

# Resume from a prior trace (payload comes from the trace)
cnips run my-pipeline --trace run_01J8X --from normalize --auto-approve

# Hot-reload a function
cnips run function webhook-handler --watch

# Run with auto-approval for approval steps
cnips run my-pipeline --payload '{}' --auto-approve --verbose
```

### 6. Push Changes

```bash
# Validate before pushing
cnips validate

# Push to workspace
cnips push
```

## Commands

### Project Management

| Command | Description |
|---------|-------------|
| `cnips init [name]` | Scaffold a new CNIPS project |
| `cnips add [type] <name>` | Create a new component or function |
| `cnips rename [type] <old> <new>` | Rename a component, function, or pipeline |
| `cnips validate` | Validate project files before push |
| `cnips fmt [path]` | Format CNIPS YAML files canonically |
| `cnips build [name] [--force]` | Build components and functions (any failed target → exit 1) |

### Authentication

| Command | Description |
|---------|-------------|
| `cnips login` | Authenticate with CNIPS (browser OIDC or token) |
| `cnips logout` | Log this folder out (the profile is removed once no folder uses it) |
| `cnips profile` | Show this folder's login, list all saved logins, or clear them |
| `cnips switch` | Switch workspace after confirming local artifacts can be discarded |

### Workspace Sync

| Command | Description |
|---------|-------------|
| `cnips pull` | Pull workspace artifacts to local files |
| `cnips push` | Apply local files to a workspace |
| `cnips discard` | Reset local cnips artifacts and sync state (keeps login accounts) |
| `cnips diff [--remote]` | Diff vs tracking base; add `--remote` for live cnips |
| `cnips status [--remote] [-v]` | Local changes vs base; `--remote` for live cnips / conflicts, `-v` for refs |
| `cnips resolve` | Bulk-resolve sync conflicts (cnips / local / both) |
| `cnips stash [push\|list\|apply\|pop\|drop]` | Save / restore / drop local-only changes (no remote fetch) |
| `cnips config [show\|set\|edit] <name>` | View or edit encrypted configuration secrets |

### Local Execution

| Command | Description |
|---------|-------------|
| `cnips run [type] <name>` | Execute a pipeline or component locally |
| `cnips run function <name> --watch` | Serve a function with hot-reload |
| `cnips trace [run-id]` | Show a pipeline run trace (pick recent when omitted) |

### Publishing

| Command | Description |
|---------|-------------|
| `cnips publish <type> <name>` | Submit a component to the marketplace |

## Command Reference

### `cnips init`

```bash
cnips init [project-name]
```

Creates a new CNIPS project with the standard folder structure, a `.vscode/` YAML schema for `component.yaml`, and an empty `cnips.lock`. If no name is provided, uses the current directory name.

Re-running `cnips init` in an already-initialized project ensures standard folders and missing scaffold files are present without overwriting existing manifests. If you are inside a subdirectory of an existing project, the CLI reports the project root and does nothing (`cnips pull`, `cnips status` and the other project commands already work from any sub-folder).

### `cnips add`

```bash
cnips add <name> --type <type> --language <lang> [--method <method>]
cnips add <type> <name> --language <lang> [...]
```

**Flags:**
- `--type, -t` — Type: `source`, `destination`, `transformation`, `approval`, `switch`, `decision`, or `function` (optional when type is the first positional argument)
- `--language, -l` — Language: `javascript`, `python`, or `go`
- `--method` — Function HTTP method: `GET`, `POST`, `PUT`, or `DELETE` (`POST` by default)
- `--description, -d` — Description for the component manifest

On success the command refreshes `cnips.lock`, prints one details line (`language go · signature fiber-v2 · template 1.4.0 · cnips.lock updated`) and indented next steps (`edit` → `cnips validate` → `cnips push`). Interactive TTY sessions also prompt for HTTP method (functions) when omitted.

Go functions always scaffold with Fiber (`fiber-v2`). Function manifests use lowercase signature versions such as `express-v3`, `fiber-v2`, and `python-v1`.

### `cnips rename`

```bash
cnips rename <type> <old-name> <new-name>
cnips rename <old-name> <new-name>
cnips rename --dry-run <type> <old-name> <new-name>
```

Preserves platform identity while moving the local directory, rewriting pipeline references, refreshing `cnips.lock`, and remapping workspace manifests.

**Types:** `pipeline`, `function`, `source`, `destination`, `transformation`, `approval`, `switch`, `decision`

When type is omitted, the old name is resolved across those types. A unique match is renamed; multiple matches must be disambiguated (interactively on a TTY).

On success it prints `✓ Renamed …`, one details line (`moved <old> -> <new> · N pipeline reference(s) updated · N workspace manifest(s) remapped`) and indented next steps (`cnips push`), in the same layout as `cnips add`. A dry run is shown as info (`› Dry run — would rename …`) and writes nothing.

**Flags:**
- `--dry-run` — Print the move and reference counts without writing

### `cnips login`

```bash
cnips login [--base-url <url>] [--token <token>] [--workspace <id>]
```

Logins are per folder (the project root, or the current directory outside a
project). In a folder without a login, a TTY run first offers your saved logins
as `<host> · <user>` plus **New login (enter base URL)**. Picking one logs in to
that instance (reusing its cached browser token) and fetches a fresh workspace
list, so the same login can serve another workspace in another folder.

The workspace is then chosen in this order:
1. `--workspace`, when given.
2. This folder's previous workspace on the same instance.
3. The workspace the project was already pulled from, when exactly one is
   found locally: `ℹ Using workspace <name> from this project's data. Run cnips
   switch to change workspaces.` This covers re-login after an expired session
   or a removed profile.
4. Otherwise an interactive picker (`default` first, so Enter selects it); a
   single workspace is picked automatically.

Success prints `✓ Logged in to <host> · workspace <name>`.

**Flags:**
- `--base-url` — CNIPS origin URL (required with `--token`; prompted on a TTY when omitted)
- `--token` — Bearer token for direct authentication (browser login when omitted)
- `--workspace` — Workspace to select (required when stdin is not a TTY)
- `--force` — Force fresh login, ignoring cached tokens

### `cnips logout`

```bash
cnips logout
```

Logs this folder out. The folder is detached from its profile; the profile
(and its cached token) is deleted only when no other folder uses it. Outside a
logged-in folder nothing changes. Run `cnips login` again in the folder; a
pulled project keeps its workspace automatically.

### `cnips profile`

```bash
cnips profile                 # this folder's login
cnips profile --all           # every saved login and its folders
cnips profile --clear         # same as cnips logout
cnips profile --clear --all   # delete every saved login (asks first)
```

Each login prints as one line, `<host> · workspace <name> · <user> · <state>`,
coloured by state: `✓` logged in, `⚠` token expired, faint when logged out.
`--clear --all` asks with a Yes/No menu on a TTY and requires an explicit `y`
when stdin is piped.

### `cnips switch`

```bash
cnips switch [--pull|--no-pull]
```

Switches the workspace for this folder's login. It must run inside a cnips
project; outside one it stops with the `Not a cnips project` hint before any
prompt. The command warns that
local cnips files will be deleted and, when the project has unpushed local
changes, says how many (`3 local change(s) not pushed — run cnips push first to
push them.`) before it asks for confirmation. It then shows only
workspaces other than the current one, resets the project to a clean init-style
layout, and updates the profile. On a TTY it then asks whether to pull the new
workspace.

**Flags:**
- `--pull` — Pull the new workspace after switching (required to pull when stdin is not a TTY)
- `--no-pull` — Skip the post-switch pull prompt

### `cnips discard`

```bash
cnips discard [--yes]
```

Removes local cnips artifacts and sync/cache state for this project, then
recreates the empty init-style project folders and resets `cnips.lock`. Login
accounts, `cnips.yaml`, `environments/`, `.git`, and unrelated repository files
are kept. In a TTY the command prompts for confirmation; non-interactive runs
must pass `--yes`.

**Flags:**
- `--yes` — Confirm discard without a prompt (required when stdin is not a TTY)

### `cnips pull`

```bash
cnips pull [--dry-run] [--skip-versions] [--accept-cnips] [--accept-local] [--write-conflicts]
```

Fetches pipelines, components, functions, global variables, and configurations from a workspace and writes them as canonical YAML files.

When local and remote both changed the same path, pull stops and offers a
conflict menu (TTY) or requires a non-interactive flag:

1. **Accept all incoming from cnips** — overwrite conflicted paths with remote
2. **Keep all local** — keep conflicted paths as-is; still apply non-conflicting incoming changes
3. **Write conflict markers** — write git-like `<<<<<<< local` / `>>>>>>> cnips` markers and resolve later
4. **Cancel** — leave the working tree unchanged

Non-conflicting remote changes can still be applied when only local-only edits exist outside conflicts. Pull also repairs a drifted tracking base when local and remote already agree. Uses the workspace and credentials from `cnips login`.

**Flags:**
- `--dry-run` — Print summary without writing files
- `--skip-versions` — Skip component version lookups
- `--accept-cnips` — On conflicts, accept all incoming cnips versions (non-interactive)
- `--accept-local` — On conflicts, keep all local versions (non-interactive)
- `--write-conflicts` — On conflicts, write conflict markers (non-interactive)

Use only one of `--accept-cnips`, `--accept-local`, or `--write-conflicts`.

### `cnips resolve`

```bash
cnips resolve [--accept-cnips|--accept-local|--write-conflicts]
```

Bulk-resolves every conflicted path after pull wrote markers or recorded conflicts.

Interactive TTY shows an arrow menu when no flag is set. Non-interactive runs
must pass exactly one of the flags below.

**Flags:**
- `--accept-cnips` — Accept incoming from cnips
- `--accept-local` — Keep current local content
- `--write-conflicts` — Keep / rewrite conflict markers

After applying, the follow-up hint reads `Next: cnips push to publish remaining local changes`.

### `cnips config`

```bash
cnips config show [name] [--reveal]
cnips config set [name] [field]
cnips config edit [name]
```

Omit `name` to pick a configuration from a list (needs a terminal). Esc cancels any prompt, including the hidden value prompt of `set`.

Configuration secrets are encrypted at rest. `cnips pull` writes the real values from the platform, but every secret field is stored as `enc:v1:<base64>` (AES-256-GCM): `spec.value` for every configuration type, and secret fields under `spec.apiAccess` (`apikey`, `totpkey`, `client_secret`, `password`, …). Other fields such as placeholders, placements and URLs stay readable, so `diff` still works.

- **Key:** 32 random bytes in `<data dir>/secret.key` (0600). The data dir is `<user config dir>/cnips/projects/<hash of project path>/`, outside the repository and never committed. `CNIPS_SECRET_KEY` (64 hex characters) overrides it for CI.
- **Stable files:** if a re-pull finds the same secret already on disk, it keeps the existing encrypted bytes, so `status` and `diff` show no change.
- **Push** decrypts in memory and sends plain text over HTTPS; the platform re-encrypts with its own key. Files from older CLI versions that still hold `platform://…` placeholders are refused (run `cnips pull` first), so a placeholder can never overwrite the real secret.
- **Plain text typed by hand** is encrypted in place by the next `cnips fmt` or `cnips push` (`⚠ Encrypted N plaintext secret(s) …`). `cnips validate` warns about it.
- **New machine or moved project:** the key is per machine and per project path, so files can't be decrypted elsewhere. Run `cnips pull` to rewrite them with the local key.

Subcommands:
- `show` — prints the configuration with secrets masked; `--reveal` prints decrypted values
- `set` — reads one secret without echo (so it never lands in shell history) and stores it encrypted. `field` is `value` or an `apiAccess` path such as `apiAccess.apikeyDetails.apikey`; omit it to pick from the secret fields
- `edit` — opens a decrypted copy in `$VISUAL` / `$EDITOR`. The copy is a 0600 file in the data dir, deleted afterwards; secrets are re-encrypted on save

### `cnips push`

```bash
cnips push [--dry-run]
cnips push <type> <name> [...]
```

Applies local canonical files to a target workspace. Creates or updates components, functions, and pipelines. Waits for server-side builds to complete.

**Interactive (no arguments, on a terminal):** push first prints how many components changed locally (or `Nothing to push` and exits), then asks:
1. **Push all** — full push
2. **Search by name** — finds the artifact; if it has no local change, says so and exits
3. **Select from changed artifacts** — checklist of changed artifacts (`↑/↓` move · `Space` toggle · `a` all · `Enter` confirm · `Esc` cancel)

Whatever is selected — one artifact or many — is pushed in **one pass**: one validate, one compare, one plan, one apply.

**Flow (git-like — push does not pull/fast-forward):**
1. Refresh `cnips.lock` (skipped with `--dry-run`)
2. Requires a local tracking base (`cnips pull` first); otherwise exits non-zero
3. If nothing in scope changed locally → `Nothing to push` (skip validate + remote)
4. Validate only the changed artifacts being pushed (any kind, plus `cnips.yaml` / `cnips.lock`); stop on errors
5. Compare with cnips:
   - Full push — remote ahead or conflicts → **reject** (exit 1; `cnips status --remote`, then `cnips pull`)
   - Scoped push — artifacts with remote changes or conflicts are **skipped** and listed (`⚠ Skipped <kind> <name>`); the rest are pushed
6. If one artifact fails mid-apply, **roll back every** create/update from this push (all-or-nothing)
7. After success, re-fetch the workspace snapshot (full push) or advance the tracking base for the pushed paths (scoped push)

Output:

```
  ✓ Validation passed for 2 components

  › Pushing to https://dev.cnips.eu
  › Workspace: default
  › Plan · 2 components · 0 create · 2 update
    KIND          ACTION   NAME
    source        update   orders-in
    switch        update   route-orders

  ✓ Pushing ████████████████████████ 100%
  ✓ Push complete · 2 components pushed (0 created · 2 updated) · 1 skipped

  Next steps:
    cnips status
```

While cnips builds, a spinner line (`waiting for cnips build: <name> (12s)`) animates in place; when stdout is not a terminal, a plain line is printed when the message changes.

Scoped push (selected artifacts, or `cnips push transformation normalize-order`): other local edits are stashed and restored silently around the push. If cnips has changes outside the selection, push asks whether to pull them afterwards. Push does not write conflict markers or call `cnips resolve` — pull/resolve first when the workspace is out of sync.

**Flags:**
- `--dry-run` — Print plan without applying changes

### `cnips status`

```bash
cnips status [--remote] [-v]
```

By default compares the working tree to the local tracking base only (no live
cnips fetch). Pass `--remote` to also show conflicts, incoming cnips changes,
and auto-mergeable paths.

**Flags:**
- `--remote` — Fetch live cnips state and include conflicts / incoming / auto-mergeable
- `-v, --verbose` — Also print the `Refs` section (Base / Local, plus CNIPS with `--remote`)

### `cnips diff`

```bash
cnips diff [--remote] [--format unified|summary] [--exit-code]
```

Shows differences between the working tree and the local tracking base. With
`--remote`, fetches live cnips state for a three-way diff.

Unified output uses standard `diff --git a/<path> b/<path>` / `--- a/<path>` / `+++ b/<path>` headers (`/dev/null` for added or deleted files), so pagers such as `delta` or `diff-so-fancy` and editors can colour it. A muted `a/ = base · b/ = local` line (or `a/ = cnips`) names the sides. On a terminal the diff is paged through `$PAGER` (default `less -FRX`, which exits straight away when the output fits on one screen); set `PAGER=cat` to turn paging off.

**Flags:**
- `--remote` — Fetch live cnips state (three-way diff)
- `--format` — `unified` (default) or `summary`
- `--exit-code` — Exit with code 1 when differences are found

### `cnips stash`

```bash
cnips stash [push|list|apply|pop|drop] [stash-id]
```

Saves local-only paths into a stash (same local-vs-base seam as `cnips status`
without `--remote`), then resets those paths to the tracking base. Does not
fetch live cnips. Use `cnips pull` for incoming changes, then `cnips stash pop`
/ `apply` to restore. `cnips stash drop [id]` removes a stash entry without
applying it. `list` prints recent stashes as aligned columns
(`STASH  WORKSPACE  FILES  CREATED`) with relative times such as `2h ago` (no pagination).

**Flags (`apply` / `pop` conflicts):**
- `--accept-stash` — Take stashed content (non-interactive)
- `--accept-local` — Keep current local content (non-interactive)
- `--write-conflicts` — Write conflict markers (non-interactive)

### `cnips run`

```bash
cnips run <pipeline> [--payload <json>|--payload-file <path>] [--env <env>]
cnips run [type] <name> [--payload <json>]
cnips run function <name> --watch
```

Executes a pipeline locally, a single local artifact, or (with `--watch`) a hot-reload HTTP server for a function.

Before running, the target is validated (for a pipeline: the pipeline plus every component its steps use); errors stop the run. Custom `switch/…` and `decision/…` components route by the label they return (matched against `cases` / `branches`, falling back to `default_label`, `default`, then `next`).

Supports native components:
- `native/transformation@1` — JSONata transformations
- `native/decision@1` — Boolean conditions with branching
- `native/switch@1` — Multi-way routing
- `native/loop@1` — Array iteration (sequential or concurrent)
- `native/approval@1` — Interactive terminal prompts
- `native/http-source@1` — Outbound HTTP calls

**Configurations in pipelines:** steps use a configuration by id, never by loading all of them:
- HTTP request (`native/http-request`): `api_access_ref: <configuration>` adds its credential the way the platform's processor does: API key in the configured header or query parameter, TOTP code (240 s period), `Authorization: Bearer <token>` from the OAuth client-credentials flow (via `wellknownUrl`), or `Authorization: Basic …`.
- Crypto `hmac` and JWT `sign` / `verify`: `secret_access_id: <configuration>` uses that configuration's key as the signing secret, decrypting `enc:v1:` values with the local project key (an environment variable of the same name is still accepted as a fallback).
- `${{ env.X }}` resolves pipeline variables, `environments/<env>.yaml` variables and global variables only. This applies to pipeline runs; standalone `cnips run component` / `cnips run function` runs do not load global or environment variables. **Configurations are no longer template variables** (same as the platform); `cnips validate` warns when a pipeline still uses `${{ env.<configuration> }}`.

**Payload:**
- New runs require `--payload` or `--payload-file` (exactly one).
- With `--trace`, the payload is taken from the prior run. Any `--payload` / `--payload-file` flags are ignored.
- `--from` is optional and requires `--trace`.

**`--watch` (functions only):**
Starts a local HTTP server with file watching. Rebuilds and restarts on source changes.

Endpoints:
- `GET /ping` — Health check
- `<method> /execute` — Method from `functions/<name>/cnips.fn.yaml` (`POST` by default)

**Secured functions (`apiAccessRef`):** when `cnips.fn.yaml` sets `spec.apiAccessRef` to a configuration (matched by its `ref` id or name), the function behaves as it does on the platform (`faas-srv`):
- `--watch` checks every request except `/ping`: the API key or TOTP code in the configured header or query parameter, or `Authorization: Bearer <token>` for OAuth. Requests without a valid credential get `401` and an access-log line such as `POST /execute  401  1ms  invalid or missing API key`. The startup banner names what is required, never the value: `Requires API key in header X-API-Key (configuration "partner-key")`.
- One-shot `cnips run function` builds the same credential itself (the key or a fresh TOTP code), checks it, sends it, and collects the logs. For OAuth2 configurations it asks for the access token instead (hidden input, like the webapp's invoke dialog) and sends it as `Authorization: Bearer <token>`, adding the `Bearer ` prefix when missing. A token is required, so the command needs a terminal.
- Locally an OAuth bearer token is only checked for presence; verifying it needs the identity provider. `KeyValue` configurations and basic auth can't secure a function (same as the platform).

**Run files:** `.data/functions/<name>/run_<id>/` is owner-only (0700, files 0600). `request.json` writes `Authorization`, `Proxy-Authorization`, `Cookie`, `X-API-Key` and any header carrying the credential as `[REDACTED]`, and known secret values are masked in `function.log` and `server.log`.

Entries under `spec.config` in `cnips.fn.yaml` are exposed as request headers when the incoming request does not already include the same header (mirrors the platform function runtime).

The server listens on a free local port; the URL is printed on startup. Each request prints one access-log line (`POST /execute  200  14ms  req 0b6c3f1e`) with that request's function logs indented below it.

**Output:** a single-event pipeline run prints one `✓`/`✗` row per step (`step   kind/name   duration`), the same row style as fan-out runs and `cnips build`, then a tally such as `✓ ok · 3 steps (0.41s)`. A failed `cnips run function` prints `✗ <HTTP status> · <duration>` (for example `✗ 500 · 0.42s`) when the runner returns a status, and `✗ failed · <duration>` otherwise.

**Flags:**
- `--type, -t` — Run one artifact instead of a pipeline (`pipeline`, `source`, `destination`, `transformation`, `approval`, `switch`, `decision`, or `function`)
- `--payload, -p` — JSON string payload (ignored when `--trace` is set)
- `--payload-file, -f` — Path to JSON payload file (ignored when `--trace` is set)
- `--env, -e` — Environment name (default: `dev`)
- `--trace` — Previous trace run ID or path (reuses payload from that run)
- `--from` — Start from a specific step ID (requires `--trace`)
- `--watch` — Hot-reload HTTP server (functions only)
- `--auto-approve` — Auto-approve approval steps
- `--auto-reject` — Auto-reject approval steps
- `--verbose, -v` — Print each step as it executes

Without `--auto-approve` / `--auto-reject`, each `native/approval` step asks Approve or Reject; Esc cancels the whole run. Runs without a terminal must pass one of the two flags.

### `cnips trace`

```bash
cnips trace [run-id] [--json]
```

Shows a saved local pipeline run trace. Pass a run id (or path to a `.json` file). With no argument on a TTY, pick from recent traces (scrollable list).

In the step-by-step event view (`--event N`) each step line is bold with a coloured `✓`/`✗`; the `input` / `output` labels are faint and errors are labelled in red. Switch and decision steps also show the `route` they took.

**Flags:**
- `--json` — Print the full trace JSON

### `cnips validate`

```bash
cnips validate [--type <type>]
```

Validates project files before push:
- YAML parsing for all artifact types
- Pipeline graph integrity and step references
- Pipeline graph rules (same as the web app's save check): exactly one source step (or `trigger.sourceRef`), at least one destination, every end node is a destination, decisions have both TRUE and FALSE branches
- Component/function reference resolution
- Secret leak detection (literal credentials in YAML)
- Lock file integrity verification
- Language requirements for coded extractors only (streaming sources such as Kafka/MQTT do not require a language)

Configuration checks: plain-text secrets (encrypted by the next `fmt` / `push`), `platform://` placeholders left by older CLI versions (run `cnips pull`), and `${{ env.<configuration> }}` references in pipelines.

Findings are grouped under the item they belong to (for example `› pipelines/orders`), with each error marked `✗` and each warning `⚠`, indented below it. The command ends with `✗ Validation failed · 2 errors · 1 warning` or `✓ Validation passed`.

`cnips validate` checks the whole project. `cnips push`, `cnips run` and `cnips build` run the same checks but report only findings for the artifacts they touch (`✓ Validation passed for N components`); `cnips.yaml` and `cnips.lock` findings always count.

### `cnips fmt`

```bash
cnips fmt [path] [--dry-run] [--type <type>]
```

Rewrites CNIPS YAML files in canonical form. With `--dry-run`, lists files that
would change (exit 1 if any).

**Flags:**
- `--dry-run` — Report files that would change without writing them
- `--type, -t` — Only format one component type

### `cnips build`

```bash
cnips build [name] [--force] [-v]
```

Builds a component or function, or every component a pipeline uses. Without a
name it prompts for one (with `--force`, builds the components of all local
pipelines). Only the targets being built are validated first. Any failed target
exits 1.

**Flags:**
- `--force` — Rebuild even when the cache digest matches
- `-v, --verbose` — Show artifact paths and full tool output

### `cnips publish`

```bash
cnips publish <type> <name> [--marketplace-version <version>]
```

Submits a pushed component to the CNIPS marketplace. Requires a successful platform build.

**Types:** `transformation`, `destination`, `switch`, `approval`, `decision`

Prints `✓ Publish submitted: <type> "<name>" v<version>` followed by `artifact <file> · request <id>`. While it waits for the marketplace upload, one spinner line updates in place.

**Flags:**
- `--component-version` — Platform version to publish (default: latest)
- `--marketplace-version` — Semantic version for marketplace (default: next patch)
- `--description` — Marketplace description
- `--logo-url` — Marketplace logo URL
- `--mask-publisher-name` — Publish using the masked publisher name
- `--wait` — Wait for upload completion (default: `true`)

## Configuration

### Project Manifest (`cnips.yaml`)

```yaml
apiVersion: cnips.io/v1
kind: Project
metadata:
  name: my-integrations
spec:
  runtime: "2026.8"
  registry: https://marketplace.cnips.io
```

### Environment Configuration (`environments/dev.yaml`)

```yaml
apiVersion: cnips.io/v1
kind: Environment
metadata:
  name: dev
spec:
  target:
    workspace: local
  variables:
    region: local
  secrets:
    api_key:
      ref: platform://secrets/api-key
```

### Auth Profiles

Logins are stored in the local user config directory, usually
`~/.config/cnips/config.json`. Each entry is one user on one instance and
workspace, with the folders that use it:

```json
{
  "logins": [
    {
      "baseUrl": "https://your-cnips-instance.com",
      "userId": "…",
      "workspaceId": "default",
      "directories": ["/home/me/project-a"]
    }
  ]
}
```

A folder without an entry has no login; use `cnips profile --all` to inspect
them.

### Environment variables

| Variable | Effect |
|----------|--------|
| `NO_COLOR` | Disable ANSI colors (community standard); also switches UI symbols to ASCII |
| `CNIPS_ASCII` | Force ASCII symbols/progress bars even when color is enabled |
| `CNIPS_NO_HISTORY` | Disable interactive prompt history persistence (`~/.config/cnips/input_history`) |
| `CNIPS_CONFIG` | Override path to the CLI auth config file |
| `CNIPS_SECRET_KEY` | 64-hex-character key for configuration secrets (CI / headless); overrides `<data dir>/secret.key` |
| `PAGER` | Pager for `cnips diff` on a terminal (default `less -FRX`; `cat` disables paging) |
| `CNIPS_DATA_DIR` | Override where per-project sync state is stored (default: `<user config dir>/cnips/projects`) |

## Examples

### Creating a JSONata Transformation

```bash
cnips add normalize-order --type transformation --language javascript
```

Edit `transformations/normalize-order/handler.js`:

```javascript
// JSONata expression for order normalization
export default `{
  "orderId": id,
  "customer": customer.name,
  "total": $sum(items.price * items.quantity),
  "items": items.{
    "sku": productId,
    "qty": quantity,
    "price": price
  }
}`;
```

### Creating a Pipeline

Create `pipelines/order-sync/pipeline.yaml`:

```yaml
apiVersion: cnips.io/v1
kind: Pipeline
metadata:
  name: order-sync
spec:
  steps:
    - id: fetch-orders
      uses: source/api-fetcher@latest
      next: normalize

    - id: normalize
      uses: transformation/normalize-order@latest
      next: route

    - id: route
      uses: native/switch@1
      with:
        expression: "$.total > 1000"
      cases:
        "true": high-value
        "false": standard

    - id: high-value
      uses: destination/priority-queue@latest

    - id: standard
      uses: destination/standard-queue@latest
```

### Running with Different Environments

```bash
# Development
cnips run order-sync --env dev --payload-file ./test/order.json

# Staging (requires environments/staging.yaml)
cnips run order-sync --env staging --payload-file ./test/order.json
```

### Function Development Workflow

```bash
# Create a Go function
cnips add webhook-handler --type function --language go

# Start hot-reload server
cnips run function webhook-handler --watch

# In another terminal, test the function
# (use the port printed by the watch server)
curl -X POST http://localhost:<port>/execute \
  -H "Content-Type: application/json" \
  -d '{"event": "order.created", "data": {"id": "123"}}'
```

### Publishing to Marketplace

```bash
# Ensure component is pushed and built
cnips push

# Publish to marketplace
cnips publish transformation normalize-order \
  --marketplace-version 1.0.0 \
  --description "Normalizes order data to canonical format"
```

### Status and pull conflicts

```bash
# Fast local status (no network)
cnips status

# Include live cnips / conflicts / auto-mergeable
cnips status --remote

# Interactive (TTY): choose accept-cnips, keep local, write markers, or cancel
cnips pull

# Non-interactive: accept all incoming from cnips on conflicted paths
cnips pull --accept-cnips

# Non-interactive: keep local on conflicted paths (still apply non-conflicts)
cnips pull --accept-local

# Non-interactive: write markers, then bulk-resolve
cnips pull --write-conflicts
cnips resolve --accept-local     # or --accept-cnips / --write-conflicts
```

## Requirements

- **Bun** (recommended) or **Node.js** for JavaScript/TypeScript functions
- **Python 3.12+** for Python functions
- Access to a **CNIPS instance** for sync and publish operations

## License

This project is licensed under the GNU General Public License v3.0 — see the LICENSE file for details.
