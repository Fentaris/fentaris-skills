# Configuration

## Setup Sequence

1. Explain the selected workflow and why it fits the user's users, machines, runtime, MCP servers, and current auth constraints.
2. Create or select the project directory.
3. Install or use the Fentaris CLI.
4. Run `fentaris init --help` and choose explicit CLI options for package manager, host, port, path, install, and git behavior instead of relying on interactive prompts.
5. Generate the project with `fentaris init <project-name> --non-interactive` plus the explicit options supported by the installed CLI. If `--non-interactive` still prompts, use documented options or ask the user for that one prompt value.
6. Inspect the generated project before editing. Prefer CLI commands and generated config over TypeScript rewrites.
7. Configure `fentaris.json` for project discovery, runtime entrypoint, port, host, endpoint path, and local auth directory.
8. Edit the TypeScript entrypoint only for the smallest supported app declaration changes:
   - `fentaris(...)` as the app boundary.
   - `app.mcp(...)` or `mcp(...)` for upstream servers.
   - `app.local(...)` for custom/local MCP servers and methods implemented inside the app.
   - `policy(...)`, `group(...)`, and `user(...)` for access control.
   - Built-in transport helpers such as stdio or Streamable HTTP where appropriate.
9. Add secrets through Fentaris encrypted secret commands with `--non-interactive`. Do not store secret values in `.env` files or raw environment variables, and do not embed secret values in code.
10. Add user API keys through `fentaris auth api-key` commands when the installed CLI is 1.1.0 or newer; do not create custom registration scripts.
11. Add docs only when they help the user run the generated project. Do not add custom scripts unless the user explicitly requested the script and no CLI command exists.

## User And Runtime Decisions

- Same-machine local use: keep `host` on `127.0.0.1` and explain that the MCP client connects locally.
- Different computers or team use: discuss where the proxy runs, which network can reach it, and what auth/policy protects it.
- Cloud-shaped setup: keep non-secret runtime config in `fentaris.json` and deploy-ready, but do not run deploy because deploy is not available yet. Keep actual secret values in Fentaris encrypted secrets.
- Existing app identity: map trusted headers or app context into Fentaris user identity at the proxy edge.
- OAuth request: explain that OAuth 2.1 is not supported yet and choose API-key/header identity or an external auth boundary instead.

## Preferred Implementation Pattern

Prefer the generated entrypoint and keep it small. Runtime config such as host, port, path, runtime entrypoint, and local auth directory must come from `fentaris.json` rather than `process.env`.

When TypeScript edits are necessary, prefer a single readable app boundary:

```ts
import { fentaris, group, policy, stdio, user } from "@fentaris/core";

const supportPolicy = policy("support")
  .mcp("github")
  .allow("create_issue");

const app = fentaris({
  groups: [
    group({
      id: "support",
      users: [user("alice")],
      policy: supportPolicy,
    }),
  ],
  autoLog: true,
});

app.mcp("github", {
  displayName: "GitHub",
  transport: stdio({ command: "github-mcp-server" }),
});

await app.start();
```

Adapt the example to the user's actual upstreams and package manager. Do not copy it blindly when the generated project already has a suitable structure.

## Custom Local MCP Capabilities

When a project needs custom methods or app-owned MCP capabilities, use `app.local(name)`. This creates a local MCP server inside Fentaris, so normal policy, middleware, events, logging, and client-visible naming apply.

```ts
import { Policy, fentaris } from "@fentaris/core";

const app = fentaris({
  policy: new Policy({ name: "workspace" }).mcp("workspace").allow("status"),
});

app.local("workspace")
  .tool("status", { inputSchema: { type: "object" } }, (ctx) => ({
    content: [{ type: "text", text: `ready:${ctx.auth.userId ?? "anonymous"}` }],
  }))
  .resource("config://current", { name: "Current config" }, (_ctx, params) => ({
    contents: [{ uri: params.uri, text: "ok" }],
  }));

await app.start();
```

Use this pattern for app-owned tools, resources, resource templates, prompts, and completions. Keep the local namespace distinct from upstream server names. Do not build a custom transport or plugin when `app.local(...)` fits.

## What Not To Generate

Avoid these patterns unless the user explicitly asks and the Fentaris CLI/docs do not already support the need:

- Manual `FentarisAuth.local(...)` setup in the app entrypoint.
- `process.env` checks for upstream API tokens in `src/index.ts`.
- `.env.example` files containing secret names as the default credential workflow.
- Custom `scripts/add-agent.ts` or hand-rolled user/API-key registration.
- Custom encrypted credential files or direct calls to encryption/decryption helpers.
- `app.on("tool:success")` / `app.on("tool:error")` logging hooks for basic setup.
- Reading compiled files under `node_modules/@fentaris/core/dist/*` to infer public API.
- Shell-wrapped MCP commands such as `sh -lc` when direct command/args are possible.

If the user asks for per-agent statistics, map that to Fentaris-supported identity/users/groups first and add API keys with `fentaris auth api-key`.

## Upstream MCP Servers

For common upstreams such as GitHub or Notion:

- Prefer documented Fentaris CLI/server-add commands if available.
- If editing TypeScript, use stable server names and direct transport declarations.
- Verify the upstream package command from official upstream docs or generated examples when it is not obvious.
- Keep upstream credentials in Fentaris encrypted secrets. In TypeScript, reference secret names through the supported Fentaris config/API; do not pull raw tokens from `process.env`.
- If the current Fentaris version cannot inject a secret into a stdio environment cleanly, stop and explain the limitation instead of building a workaround with shell interpolation.

## Fentaris Config

Use `fentaris.json` as the source of truth for non-secret project and runtime configuration.

- Put project discovery, runtime entrypoint, host, port, endpoint path, and local auth directory in `fentaris.json`.
- Do not generate `process.env` fallbacks for host, port, or endpoint path as the default setup pattern.
- Keep per-environment non-secret values in the Fentaris config model supported by the current CLI/docs, not in ad hoc env vars.
- Keep secret values out of `fentaris.json`; store them as Fentaris encrypted secrets and reference them by name when the config/API supports it.

## Encrypted Secrets

Use Fentaris CLI secret management as the default path for credentials.

- Discover the installed command shape with `fentaris secrets --help` when needed.
- Use the documented secret set/update command with `--non-interactive` and explicit inputs.
- Store references or secret names in project configuration, not plaintext values.
- Do not create `.env` files for secret values unless the user explicitly rejects Fentaris encrypted secrets and accepts the tradeoff.
- Do not echo, log, document, or commit secret values.
- If a secret value is required and the user has not provided it through a safe path, stop and ask for the value or for permission to leave a placeholder secret name.

## User API Keys

For Fentaris CLI 1.1.0 and newer, user API keys are CLI-managed local auth state:

- Add a provided key with `fentaris auth api-key add <user-id> --value-stdin`.
- Generate a new key with `fentaris auth api-key add <user-id> --generate`; record it for the user only once because Fentaris stores a hash.
- List stored key counts with `fentaris auth api-key list --user <user-id> --json` when automation needs structured output.
- Remove a key with `fentaris auth api-key remove <user-id> --value-stdin`.
- Clients authenticate with `x-fentaris-api-key`.

Prefer `--value-stdin` over `--value` because command arguments can expose secrets. Use `--key` or `FENTARIS_AUTH_KEY` only according to the project's local secrets backend setup, and never print user-provided key values.

## Validation

Run the narrowest meaningful checks first:

- `fentaris check --offline` for static project checks.
- `fentaris doctor` for project/environment diagnostics.
- `fentaris doctor --runtime` after the proxy is running.
- Existing `build`, `typecheck`, or `test` scripts.

Use `--non-interactive` for Fentaris CLI validation and setup commands when supported. If a command prompts for a secret or production decision, ask the user.

## Documentation Lookup

Use local docs when inside the Fentaris repository. Otherwise, consult `https://fentaris.mintlify.app` for current CLI commands, config fields, and API examples when unsure.
