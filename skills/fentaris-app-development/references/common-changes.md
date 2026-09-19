# Common Changes

## Add an Upstream MCP Server

- Preserve stable server names because they affect exposed tool names.
- Prefer a high-level `app.mcp("name", { transport })` or `mcp("name", { transport })` declaration.
- Add display metadata when useful for logs and client diagnostics.
- Update policy to grant only the intended tools.

## Add Or Change Upstream OAuth 2.1

- First distinguish upstream OAuth from client authentication at the Fentaris endpoint.
- Use `auth: oauth()` only with native `streamableHttp()` or `sse()` transports. Stdio cannot carry this authorization.
- Use `oauth()` for discovery, dynamic registration, PKCE, refresh, and per-user tokens; use preregistered options when the provider requires a client id; use `oauth.clientCredentials(...)` only for an intentional machine identity.
- Resolve OAuth client secrets through `credential(...)`; never place a client secret literal in source.
- Keep per-user tokens when callers resolve to distinct authenticated Fentaris users. Unauthenticated callers collapse to a shared OAuth session; require explicit approval for that account sharing.
- Require `FENTARIS_AUTH_KEY` or a configured `oauth.store` when authorization must survive restarts.
- Start human consent with `fentaris auth login <mcp> --as user:<id>` and verify with `fentaris auth status [mcp] --json`. Never print tokens.
- Set `oauth.publicUrl` when the callback is behind a reverse proxy or must be externally reachable.
- Private, loopback, and link-local upstream/OAuth endpoints are rejected by default. Require explicit approval and prefer `network.allowedPrivateHosts`; use broad `allowPrivateNetworkUrls: true` only when its wider SSRF exposure is accepted.

## Add Custom Local MCP Capabilities

- Use `app.local("name")` to create an app-owned local MCP server for custom tools, resources, prompts, and completions.
- Keep local namespace names stable and distinct from upstream MCP server names.
- Implement handlers with the provided `ProxyContext` plus MCP params, and return standard MCP SDK result shapes.
- Add policy entries for the local namespace, for example `policy.mcp("workspace").allow("status")`.
- Prefer this over hand-rolled transports, custom credential stores, or plugin experiments for ordinary app-owned custom methods.

## Change Policy

- Prefer durable declarations with `policy(...)`, `group(...)`, and `user(...)`.
- Keep deny-by-default behavior intact.
- Use middleware for runtime checks based on request args, tenant, environment, or external state.
- Add or update tests for policy changes that affect access.

## Add Request Identity

- Resolve user, tenant, trace, and environment at the proxy edge.
- Keep identity mapping in one place so logging, policy, and upstream env injection use the same context.
- Avoid trusting headers unless the app is behind a known auth boundary.

## Add Middleware or Hooks

- Keep cross-cutting behavior near proxy setup unless the app already has a module convention.
- Use server/tool scoped middleware for targeted behavior.
- Avoid mutating request arguments unless the behavior is documented and tested.

## Add Secrets

- Use the project's Fentaris secret workflow or existing secret manager.
- Update secret manifests or env examples without committing secret values.
- Run secret diagnostics when available.

## Add User API Keys

- For Fentaris CLI 1.1.0 and newer, use `fentaris auth api-key` commands for local user API keys.
- Add a key with `fentaris auth api-key add <user-id> --value-stdin` when the value arrives through protected input. `--generate` prints the raw key once; run it only in a verified non-recorded secret-output channel or private human terminal.
- List keys with `fentaris auth api-key list --user <user-id> --json` when automation needs machine-readable output.
- Remove a key with `fentaris auth api-key remove <user-id> --value-stdin`.
- Clients authenticate with the `x-fentaris-api-key` header; Fentaris resolves the key to the configured user identity.
- Do not create custom `scripts/add-agent.ts`, plaintext API-key files, or direct encryption helper calls for user API-key registration.

## Endpoint Changes

- Keep `path` stable for clients, usually `/mcp`.
- Prefer config/env changes over hardcoded host and port changes.
- Use `host: "0.0.0.0"` only when exposure is intentional and auth/policy are configured.
