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
- **Works offline** (no saved token): `init`, `login`, `logout`, `discard`, `add`, `rename`, `fmt`, `validate`, `build`, `run`, `trace`, `stash`, local `status` / `diff`, plus `help`, `completion`, and `version`.
- **Exit codes:** Command failures that already printed a themed error (validate, `fmt --dry-run`, push reject, build target errors, failed pipeline `run`, `diff --exit-code`) exit **1**. User cancel on confirm/menus exits **0**.
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
| `cnips logout` | Clear the saved token for the current profile (workspace config kept) |
| `cnips switch` | Switch workspace after confirming local artifacts can be discarded |

### Workspace Sync

| Command | Description |
|---------|-------------|
| `cnips pull` | Pull workspace artifacts to local files |
| `cnips push` | Apply local files to a workspace |
| `cnips discard` | Reset local cnips artifacts and sync state (keeps login accounts) |
| `cnips diff [--remote]` | Diff vs tracking base; add `--remote` for live cnips |
| `cnips status [--remote]` | Local changes vs base; add `--remote` for live cnips / conflicts |
| `cnips resolve` | Bulk-resolve sync conflicts (cnips / local / both) |
| `cnips stash [push\|list\|apply\|pop\|drop]` | Save / restore / drop local-only changes (no remote fetch) |

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

Re-running `cnips init` in an already-initialized project ensures standard folders and missing scaffold files are present without overwriting existing manifests. If you are inside a subdirectory of an existing project, the CLI reports the project root instead of scaffolding again.

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

On success the command refreshes `cnips.lock` and prints next-step hints (`edit` → `validate` → `push`). Interactive TTY sessions also prompt for HTTP method (functions) when omitted.

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

**Flags:**
- `--dry-run` — Print the move and reference counts without writing

### `cnips login`

```bash
cnips login [--base-url <url>] [--token <token>] [--workspace <id>]
```

On first login, when no workspace is saved in the local config, the CLI shows
an interactive workspace picker (arrow keys / numbered quick pick). The
`default` workspace is listed first so Enter selects it. After `cnips logout`,
the next login keeps the existing workspace selection and refreshes only the
token. To change the workspace, use `cnips switch`.

**Flags:**
- `--base-url` — CNIPS origin URL (required with `--token`; prompted on a TTY when omitted)
- `--token` — Bearer token for direct authentication (browser login when omitted)
- `--workspace` — Workspace to select (required when stdin is not a TTY)
- `--force` — Force fresh login, ignoring cached tokens

### `cnips logout`

```bash
cnips logout
```

Clears token fields from the selected auth profile while keeping the base URL,
tenant, workspace list, and selected workspace. Commands that contact cnips
and project commands require `cnips login` again before they can use the saved
profile.

### `cnips switch`

```bash
cnips switch [--pull|--no-pull]
```

Switches the selected workspace for the current profile. The command warns that
local cnips artifacts will be discarded, asks for confirmation, shows only
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

### `cnips push`

```bash
cnips push [--dry-run]
cnips push <type> <name> [...]
```

Applies local canonical files to a target workspace. Creates or updates components, functions, and pipelines. Waits for server-side builds to complete.

**Flow (git-like — push does not pull/fast-forward):**
1. Refresh `cnips.lock` (skipped with `--dry-run`)
2. Requires a local tracking base (`cnips pull` first); otherwise exits non-zero
3. If local matches the tracking base → `Nothing to push` (skip validate + remote)
4. Validate the local tree; stop on errors
5. Compare with cnips:
   - Full push — remote ahead or conflicts → **reject** (exit 1; `cnips status --remote`, then `cnips pull`)
   - Scoped push — remote ahead or conflicts on that artifact → **skip** (exit 0; pull first)
6. If one artifact fails mid-apply, **roll back every** create/update from this push (all-or-nothing)
7. After success, re-fetch the workspace snapshot (updates lock + tracking base)

Scoped form (`cnips push transformation normalize-order`) gates and uploads only that artifact. Other local edits are stashed, the workspace is re-pulled after push, then the stash is popped so those edits return as local-only. Push does not write conflict markers or call `cnips resolve` — pull/resolve first when the workspace is out of sync.

**Flags:**
- `--dry-run` — Print plan without applying changes

### `cnips status`

```bash
cnips status [--remote]
```

By default compares the working tree to the local tracking base only (no live
cnips fetch). Pass `--remote` to also show conflicts, incoming cnips changes,
and auto-mergeable paths, plus the CNIPS ref digest.

**Flags:**
- `--remote` — Fetch live cnips state and include conflicts / incoming / auto-mergeable

### `cnips diff`

```bash
cnips diff [--remote] [--format unified|summary] [--exit-code]
```

Shows differences between the working tree and the local tracking base. With
`--remote`, fetches live cnips state for a three-way diff.

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
applying it. `list` prints recent stashes (no pagination).

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

Supports native components:
- `native/transformation@1` — JSONata transformations
- `native/decision@1` — Boolean conditions with branching
- `native/switch@1` — Multi-way routing
- `native/loop@1` — Array iteration (sequential or concurrent)
- `native/approval@1` — Interactive terminal prompts
- `native/http-source@1` — Outbound HTTP calls

**Payload:**
- New runs require `--payload` or `--payload-file` (exactly one).
- With `--trace`, the payload is taken from the prior run. Any `--payload` / `--payload-file` flags are ignored.
- `--from` is optional and requires `--trace`.

**`--watch` (functions only):**
Starts a local HTTP server with file watching. Rebuilds and restarts on source changes.

Endpoints:
- `GET /ping` — Health check
- `<method> /execute` — Method from `functions/<name>/cnips.fn.yaml` (`POST` by default)

Entries under `spec.config` in `cnips.fn.yaml` are exposed as request headers when the incoming request does not already include the same header (mirrors the platform function runtime).

The server listens on a free local port; the URL is printed on startup.

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

### `cnips trace`

```bash
cnips trace [run-id] [--json]
```

Shows a saved local pipeline run trace. Pass a run id (or path to a `.json` file). With no argument on a TTY, pick from recent traces (scrollable list).

**Flags:**
- `--json` — Print the full trace JSON

### `cnips validate`

```bash
cnips validate [--type <type>]
```

Validates project files before push:
- YAML parsing for all artifact types
- Pipeline graph integrity and step references
- Component/function reference resolution
- Secret leak detection (literal credentials in YAML)
- Lock file integrity verification
- Language requirements for coded extractors only (streaming sources such as Kafka/MQTT do not require a language)

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
cnips build [name] [--force]
```

Builds local components and functions (all when no name is given). Any failed
target exits 1.

**Flags:**
- `--force` — Rebuild even when the cache digest matches

### `cnips publish`

```bash
cnips publish <type> <name> [--marketplace-version <version>]
```

Submits a pushed component to the CNIPS marketplace. Requires a successful platform build.

**Types:** `transformation`, `destination`, `switch`, `approval`, `decision`

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

Authentication profiles are stored in the local user config directory, usually
`~/.config/cnips/config.json`:

```json
{
  "currentProfile": "default",
  "profiles": {
    "default": {
      "apiUrl": "https://your-cnips-instance.com",
      "workspaceId": "default",
      "tenantKey": "my-tenant"
    }
  }
}
```

### Environment variables

| Variable | Effect |
|----------|--------|
| `NO_COLOR` | Disable ANSI colors (community standard); also switches UI symbols to ASCII |
| `CNIPS_ASCII` | Force ASCII symbols/progress bars even when color is enabled |
| `CNIPS_NO_HISTORY` | Disable interactive prompt history persistence (`~/.config/cnips/input_history`) |
| `CNIPS_CONFIG` | Override path to the CLI auth config file |
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
