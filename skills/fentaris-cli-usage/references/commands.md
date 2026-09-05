# Fentaris CLI Command Reference

Usage root: `fentaris [OPTIONS] [COMMAND]`

Global options:

- `--help` / `-h` — print help for the current command level.
- `--version` / `-v` — print the installed CLI version.
- `--non-interactive` — fail instead of prompting for input. Use for automation and agent-driven runs.

Global environment variables:

- `FENTARIS_AUTH_KEY` — encryption key used by the local secrets backend (`auth`/`secrets` commands). Prefer this over `--key` in automation.
- `FENTARIS_EDGE_STATE_DIR` — absolute directory for local Edge identity and runtime state.

Command groups:

- **Project**: `init`, `dev`, `build`
- **Health**: `check`, `doctor`
- **Secrets/identity/discovery**: `auth`, `secrets`, `tools`, `edge`

---

## `fentaris init`

Create a new Fentaris project.

```
fentaris init [OPTIONS] [project-name]
```

Creates a project template, installs dependencies unless disabled, optionally initializes git, and runs diagnostics.

Arguments:

- `project-name` — directory and package name for the new project. Optional; omit to be prompted (unless `--non-interactive`).

Options:

- `--package-manager <PM>` — package manager written to the generated project. Supported: `pnpm`, `npm`, `bun`.
- `--skip-install` — skip dependency installation.
- `--skip-git` — skip git repository initialization.
- `--port <PORT>` — port written to `fentaris.json`. Default: `4000`.
- `--path <PATH>` — MCP route path written to `fentaris.json`. Default: `/mcp`.
- `--core-version <RANGE>` — version range for `@fentaris/core` in the generated `package.json`. Accepts semver ranges (`^3.0.0`), dist tags (`latest`), and workspace/file references (`workspace:*`, `file:../packages/core`). Default: `^3.0.0`.
- `--non-interactive` — fail instead of prompting for missing project inputs.
- `--help` / `-h`

Example (agent-safe, reproducible):

```bash
fentaris init my-proxy --non-interactive --package-manager pnpm --port 4100 --path /mcp
```

The generated project listens on `http://localhost:<port>/mcp` (default `4000`).

---

## `fentaris dev`

Run the discovered project in development mode.

```
fentaris dev [OPTIONS]
```

No project-specific options; only `--help`/`-h`. Run this from inside a generated/embedded project directory (one with `fentaris.json`).

---

## `fentaris build`

Build a deterministic local artifact.

```
fentaris build [OPTIONS]
```

No project-specific options; only `--help`/`-h`. This is the closest thing to a "release" step today — there is no `fentaris deploy` command. Treat the artifact as ready to hand to your own hosting/process-manager setup.

---

## `fentaris check`

Run project checks (static, no live services required unless requested).

```
fentaris check [OPTIONS]
```

Options:

- `--offline` — skip checks that require local external services.
- `--strict` — treat warnings as failures (nonzero exit).
- `--json` — output project checks as JSON (`{ "results": [...] }`).
- `--verbose` — list passed checks in addition to issues.
- `--help` / `-h`

Exit behavior: throws/exits nonzero if any check failed, or if `--strict` is set and any check warned.

Typical agent invocation:

```bash
fentaris check --offline --json
```

---

## `fentaris doctor`

Run environment and project diagnostics.

```
fentaris doctor [OPTIONS]
```

Options:

- `--fix` — apply available automatic fixes.
- `--strict` — treat warnings as failures.
- `--json` — output diagnostics as JSON.
- `--verbose` — list passed checks in addition to issues.
- `--runtime` — include runtime connectivity checks (use after the proxy is started).
- `--timeout <MS>` — runtime check timeout in milliseconds. Default: `10000`.
- `--help` / `-h`

Exit behavior: same pass/fail/strict semantics as `check`.

Typical sequence: `fentaris check --offline` before starting the proxy, then `fentaris doctor --runtime` once it is running.

---

## `fentaris auth`

Manage local identity authentication.

```
fentaris auth [OPTIONS] [COMMAND]
```

Omit the subcommand to open an interactive menu for adding, listing, or removing local API keys (not for automation — use explicit subcommands there).

Options: `--key <KEY>` (use an explicit local encryption key instead of `FENTARIS_AUTH_KEY` or an interactive prompt; prefer `FENTARIS_AUTH_KEY` for automation), `--help`/`-h`.

### `fentaris auth api-key`

Manage API keys for local user identity. API keys authenticate clients through the `x-fentaris-api-key` header and resolve them to Fentaris users.

```
fentaris auth api-key [OPTIONS] [COMMAND]
```

#### `fentaris auth api-key add [user-id]`

Store a local API key for a user. Omit `user-id` or the value to use guided setup (user selection, generate-or-enter, redacted review, confirmation).

```
fentaris auth api-key add [OPTIONS] [user-id]
```

- `--value <VALUE>` — API key value. Prefer `--value-stdin` to avoid exposing keys in process arguments.
- `--value-stdin` — read the API key value from stdin.
- `--generate` — generate a new API key and print it once.
- `--key <KEY>` — local encryption key override.
- `--help` / `-h`

Agent-safe patterns:

```bash
# generated key, non-interactive
fentaris auth api-key add alice --generate

# caller-provided key via stdin (never via --value in scripts)
printf '%s' "$THE_KEY" | fentaris auth api-key add alice --value-stdin
```

#### `fentaris auth api-key list`

List local API-key counts by user.

```
fentaris auth api-key list [OPTIONS]
```

- `--user <ID>` — only list keys for one user id.
- `--json` — output API-key references as JSON.
- `--key <KEY>`
- `--help` / `-h`

#### `fentaris auth api-key remove <user-id>`

Remove a local API key from a user.

```
fentaris auth api-key remove [OPTIONS] <user-id>
```

- `--value <VALUE>` — API key value to remove. Prefer `--value-stdin`.
- `--value-stdin`
- `--key <KEY>`
- `--help` / `-h`

---

## `fentaris secrets`

Manage local credentials and secret manifests.

```
fentaris secrets [OPTIONS] [COMMAND]
```

Subcommands: `set`, `setup`, `list`, `unset`, `manifest`, `doctor`.

### `fentaris secrets setup`

Discover and configure all required project credentials in one pass. Generates missing Fentaris API keys, prompts for external values in interactive mode, and writes only after the full setup plan is complete. JSON and non-interactive runs never prompt and make no changes while required external values are unavailable.

```
fentaris secrets setup [OPTIONS]
```

- `--entrypoint <PATH>` — entrypoint to scan instead of the configured project entrypoint.
- `--dry-run` — show the setup plan without creating keys or changing files.
- `--yes` — apply the setup plan without confirmation.
- `--json` — output the canonical machine-readable setup envelope.
- `--key <KEY>`
- `--help` / `-h`

Agent-safe pattern: `fentaris secrets setup --dry-run --json` to inspect the plan first, then `--yes --json` to apply it once all required external values are available.

### `fentaris secrets set [reference]`

Store a local credential value. Omit `reference` or `--value` to use guided setup (manifest reference selection, scope selection, redacted review, confirmation).

```
fentaris secrets set [OPTIONS] [reference]
```

- `reference` — secret reference to store, e.g. `github.token`.
- `--user <ID>` — store the credential for a user scope.
- `--group <ID>` — store the credential for a group scope.
- `--value <VALUE>` — prefer `--value-stdin` instead.
- `--value-stdin`
- `--key <KEY>`
- `--help` / `-h`

```bash
printf '%s' "$GITHUB_TOKEN" | fentaris secrets set github.token --value-stdin
```

### `fentaris secrets list`

List required and stored credentials.

```
fentaris secrets list [OPTIONS]
```

- `--json`
- `--key <KEY>`
- `--help` / `-h`

### `fentaris secrets unset <reference>`

Remove a local credential value.

```
fentaris secrets unset [OPTIONS] <reference>
```

- `--user <ID>` / `--group <ID>` — scope to remove from.
- `--key <KEY>`
- `--help` / `-h`

### `fentaris secrets manifest`

Generate or check the secrets manifest.

```
fentaris secrets manifest [OPTIONS]
```

- `--entrypoint <PATH>` — entrypoint to scan when no `fentaris.json` is present or when overriding project config.
- `--check` — fail if `secrets.manifest.json` is missing or out of date (use in CI).
- `--help` / `-h`

### `fentaris secrets doctor`

Run secret-specific diagnostics.

```
fentaris secrets doctor [OPTIONS]
```

- `--strict`
- `--json`
- `--key <KEY>`
- `--help` / `-h`

---

## `fentaris tools`

Discover effective MCP tools for configured accounts.

```
fentaris tools [OPTIONS] [COMMAND]
```

Subcommands: `list`, `search`, `get`, `schema`, `auth`.

Shared discovery options across `list`/`search`/`get`/`schema` (`toolDiscoveryOptions`):

- `--json` — output a JSON envelope.
- `--compact` — return compact metadata.
- `--limit <N>` — maximum number of tools to return. Default: `20`.
- `--cursor <CURSOR>` — pagination cursor from a prior response.
- `--max-tokens <N>` — best-effort output token budget.
- `--mcp <MCP>` — filter to one MCP server.
- `--as <SELECTOR>` — use a configured account selector such as `user:alice` or `group:support`.
- `--include <TEXT>` — only include tools matching text (comma-separated values accepted).
- `--exclude <TEXT>` — exclude tools matching text (comma-separated values accepted).
- `--refresh` — bypass cached discovery data where supported.
- `--no-start` — do not start stdio MCP servers for discovery.
- `--help` / `-h`

### `fentaris tools list`

List effective MCP tools. `fentaris tools list [OPTIONS]` with the shared discovery options above.

### `fentaris tools search <query>`

Search effective MCP tools. `fentaris tools search [OPTIONS] <query>`.

### `fentaris tools get <tool>`

Inspect one effective MCP tool. `fentaris tools get [OPTIONS] <tool>` — `tool` is the proxied tool name, e.g. `github__create_issue`.

### `fentaris tools schema <tool>`

Inspect one effective MCP tool schema. `fentaris tools schema [OPTIONS] <tool>` plus:

- `--input` — return the input schema.
- `--output` — return the output schema.

### `fentaris tools auth`

Inspect tool account authentication.

```
fentaris tools auth [OPTIONS] [COMMAND]
```

Subcommands, each taking `--mcp <MCP>` (configured MCP server name), `--as <SELECTOR>` (configured account selector), `--json`, `--help`/`-h`:

- `fentaris tools auth list` — list configured MCP account selectors.
- `fentaris tools auth status --mcp <MCP> --as <SELECTOR>` — inspect one MCP account selector.
- `fentaris tools auth login --mcp <MCP> --as <SELECTOR>` — start or describe login for one selector.

---

## `fentaris edge` (alpha / preview)

Join, inspect, and operate governed Edge computers. Join once, run persistently when supported, and manage only devices visible to the selected Fentaris identity.

> Validate service lifecycle and recovery on every target OS before rollout; protocol, CLI, and local state formats may change before stable release.

```
fentaris edge [COMMAND]
```

Shared JSON options where noted (`edgeJsonOptions`): `--json` (canonical JSON envelope), `--verbose` (additional human-readable diagnostics), `--help`/`-h`.

Shared discovery options (`edgeDiscoveryOptions`, used by `list`/`get`/`status`): `--compact`, `--limit <COUNT>` (1-100), `--cursor <CURSOR>`, `--include <FIELDS>`, `--exclude <FIELDS>`, `--as <IDENTITY>` (`user:<name>` or `group:<name>`), plus the shared JSON options.

### `fentaris edge join <control-plane-url>`

Enroll this computer and configure persistent operation.

```
fentaris edge join [OPTIONS] <control-plane-url>
```

- `control-plane-url` (required) — HTTPS control-plane URL.
- `--name <NAME>` — stable tenant-scoped public device name.
- `--description <TEXT>`
- `--tag <TAG>` (repeatable) — add a descriptive tag.
- `--service` — require persistent service installation.
- `--no-service` — enroll without installing a persistent service.
- shared JSON options.

```bash
fentaris edge join https://control.example --name 'Mac Studio' --tag xcode --json
```

### `fentaris edge approve <user-code>`

Approve an exact pending Edge authorization through the protected local operator channel.

```
fentaris edge approve [OPTIONS] <user-code>
```

- `user-code` (required) — exact short-lived code displayed by the joining Edge.
- `--subject <SUBJECT>` (required) — Fentaris subject receiving the device grant.
- `--tenant <TENANT>` — tenant of the pending authorization. Default: `default`.
- `--actor <ACTOR>` — auditable local operator identity. Default: current OS user.
- `--yes` — confirm this exact approval without prompting.
- shared JSON options.

```bash
fentaris edge approve ABCD-EFGH --subject alice --tenant default --yes --json
```

### `fentaris edge run`

Run the enrolled Edge agent in the foreground. `fentaris edge run [OPTIONS]` — shared JSON options only.

### `fentaris edge service <install|start|stop|restart|uninstall>`

Manage the local persistent Edge service.

```
fentaris edge service <install|start|stop|restart|uninstall> [OPTIONS]
```

Each subcommand takes only the shared JSON options.

### `fentaris edge list`

List policy-visible Edge devices. `fentaris edge list [OPTIONS]` with the shared discovery options.

```bash
fentaris edge list --as user:alice --compact --limit 20 --json
```

### `fentaris edge get <device>`

Inspect one policy-visible Edge device. `fentaris edge get [OPTIONS] <device>` — `device` is the public device name; shared discovery options.

### `fentaris edge status [device]`

Show local or policy-visible remote Edge status. `fentaris edge status [OPTIONS] [device]` — omit `device` for the local installation; shared discovery options.

### `fentaris edge installation <status|review|approve|deny|retry|revoke|cleanup> [deployment-id]`

Review and operate managed MCP installations through the protected local Edge channel.

```
fentaris edge installation <status|review|approve|deny|retry|revoke|cleanup> [deployment-id] [OPTIONS]
```

- `status` — show separated installation, setup, workload, and readiness state. `deployment-id` optional.
- `review` — display bounded exact installer review material. `deployment-id` required.
- `approve` / `deny` — approve/deny the exact current installer plan locally. `deployment-id` required, plus `--yes`.
- `retry` — retry one retryable failed installation with a new attempt. `deployment-id` required, plus `--yes`.
- `revoke` — revoke local installation approval and stop dependent workloads. `deployment-id` required, plus `--yes`.
- `cleanup` — remove managed artifacts; custom external cleanup needs separate approval. `deployment-id` required, plus `--yes`.
- `review` / `approve` / `deny` also accept `--cleanup` to target the separately reviewed custom cleanup plan.
- shared JSON options apply to all.

```bash
fentaris edge installation status --json
fentaris edge installation review filesystem --json
fentaris edge installation approve filesystem --yes --json
```

### `fentaris edge update <device>`

Update user-managed Edge device metadata.

```
fentaris edge update [OPTIONS] <device>
```

- `device` (required) — public device name.
- `--expected-version <VERSION>` (required for optimistic updates) — current inventory version.
- `--name <NAME>` — new public device name.
- `--description <TEXT>` — new description.
- `--tag <TAG>` (repeatable) — replaces tags with this repeatable set.
- shared JSON options.

### `fentaris edge disconnect <device>`

Disconnect a device without revoking its identity. `fentaris edge disconnect [OPTIONS] <device>` plus `--yes` and shared JSON options.

### `fentaris edge revoke <device>`

Revoke a device identity. `fentaris edge revoke [OPTIONS] <device>` plus `--yes` and shared JSON options.
