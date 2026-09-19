# Machine Setup Workflow

## 1. Inventory Before Mutation

Inspect without changing the machine:

- operating system, architecture, shell, and whether the session can prompt graphically;
- `node`, `npm`, `npx`, supported package managers, and Git;
- `fentaris --version` and `fentaris --help` when installed;
- detected MCP-capable clients and only their documented/user-visible config locations;
- MCP entries by name and transport, with command arguments and URLs redacted where they contain sensitive values;
- installed agent skills and client-owned agent/persona definitions.

Do not recursively search the entire home directory or read unrelated browser/keychain data. Prefer client CLIs, documented paths, and known workspace config files. Report secret presence as a boolean or reference name, never a value.

## 2. Decision Handshake

Ask one focused question at a time. Offer only detected choices.

1. Which clients should use Fentaris?
2. Which discovered MCP servers should be migrated?
3. Should all official Fentaris skills be installed for each selected supported coding agent?
4. If agent/persona definitions exist, should they remain unchanged or be configured to use the selected client's Fentaris endpoint? Explain that they do not become Fentaris objects.
5. Ask auth, sharing, and autostart questions only when the selected topology needs them.

Do not ask for facts already discovered. Use safe local defaults without asking when they do not change a trust boundary: user-owned project directory, loopback host, `/mcp`, and an available port.

## 3. Prerequisites And CLI

Use the platform's supported installer and prefer a current supported Node release. Node 24 LTS aligns with current Fentaris development and is a safe default for a new machine. Do not replace a compatible user-managed Node installation merely to standardize it.

After installation, verify:

```bash
node --version
npm --version
npm install -g @fentaris/cli
fentaris --version
fentaris --help
```

Ask before elevation or system package-manager changes. Do not curl an unaudited shell script into a privileged shell.

Install skills with explicit targets discovered from `npx skills --help`/repository listing:

```bash
npx skills add Fentaris/fentaris-skills -g -a <agent> --skill '*'
```

A client may need a restart to discover newly installed skills. Continue the setup using CLI help and official docs; report the restart at the end.

## 4. Project Creation

If no suitable Fentaris project exists, inspect `fentaris init --help` and generate one with explicit options. Default the whole-machine proxy to the user-owned `~/fentaris-proxy` directory:

```bash
fentaris init "$HOME/fentaris-proxy" --non-interactive --package-manager npm --port 4000 --path /mcp
```

Choose a free port when 4000 is occupied. Keep the default host on `127.0.0.1`. Inspect generated files before adding migrated upstreams. Keep supported project settings in `fentaris.json`; `host` is not one of its fields, so configure a non-loopback binding only through the supported application option after explicit approval.

Use a development allow-all policy only for the first local validation. Replace it with an allow-list before network exposure or shared use.

## 5. Credentials And Auth

Three concerns are separate:

1. Client-to-Fentaris identity: optional for a personal loopback setup; use Fentaris API keys, trusted identity headers, or an existing boundary when needed.
2. Static upstream credentials: use Fentaris encrypted secrets and stdin-safe CLI commands.
3. Upstream OAuth 2.1: use `oauth()` on native Streamable HTTP/SSE transports, with encrypted persistence and human consent for authorization-code flows.

Per-user OAuth is valid only when each caller resolves to a distinct authenticated Fentaris user. Unauthenticated callers collapse to the shared OAuth session. For a personal unauthenticated loopback setup, explain and explicitly choose shared authorization; for multiple people or identities, configure client identity before per-user OAuth.

Never migrate a secret by embedding it in TypeScript or `fentaris.json`. If a client config contains plaintext, do not repeat it. Move it through a protected input channel or ask the human to re-enter it. Never turn a user-scoped credential into a shared credential without explicit approval.

`fentaris auth api-key add <user> --generate` prints the raw key to stdout once. Do not run it through an agent terminal whose transcript/output is retained. Use a verified secret-output channel, ask the human to generate it in a private terminal, or accept a value through protected input and pipe it to `--value-stdin`.

## 6. Validation Gate

Before any client cutover:

```bash
fentaris check --offline --json
fentaris doctor --json
```

Run the project's build/typecheck scripts, start the proxy in a supervised foreground/background process, then run:

```bash
fentaris doctor --runtime --json
fentaris tools list --json --compact
```

For OAuth, verify with `fentaris auth status [mcp] --json`. Exercise one non-destructive tool call when feasible. A server that cannot start, authenticate, or list tools remains unmigrated.

## 7. Client Cutover And Rollback

For each selected client:

1. Create a timestamped backup beside the original or in a user-approved backup directory.
2. Validate the current config syntax before editing.
3. Add a single server named `fentaris` pointing to the validated Streamable HTTP endpoint.
4. Add `x-fentaris-api-key` only through the client's supported secret mechanism.
5. Validate syntax after editing and restart/reload only when required.
6. Confirm the client can initialize and list proxied tools.
7. Ask whether migrated original entries should remain, be disabled, or be removed. Do not decide this automatically.

Rollback restores the exact backup, reloads the client, and stops/removes only Fentaris processes or user-level services created by this run. Never remove unrelated runtimes or client data.

## 8. Completion Report

Report:

- installed versions;
- project path and endpoint;
- selected clients and whether reload is required;
- migrated, skipped, and blocked MCP servers;
- auth mode and OAuth session status without secret material;
- skill targets installed;
- validation commands and results;
- backup paths;
- start/stop and rollback instructions;
- any unverified surface.
