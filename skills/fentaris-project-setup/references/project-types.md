# Project Types

Use these shapes to guide setup questions and implementation.

Read `discovery.md` first when the user is still deciding how Fentaris should fit their workflow.

## Local Developer Proxy

Choose this when the user wants a quick local MCP proxy for one developer or a prototype.

- Use `fentaris init <name>` when available.
- Keep the default loopback binding on `127.0.0.1`; keep path `/mcp` and the generated default port in `fentaris.json` unless the user asks otherwise. `host` is not a `fentaris.json` field.
- Keep policies explicit even for demos. If temporary broad access is needed, label it as development-only in code comments or local docs.
- Validate with `fentaris check --offline` and, after starting the proxy, `fentaris doctor --runtime`.

## Internal Team Proxy

Choose this when a team shares multiple upstream MCP servers through one endpoint.

- Use stable server names because they become client-visible prefixes.
- Add users/groups and policies early.
- Add Fentaris-managed API keys with `fentaris auth api-key` when local user identity is needed.
- Add request identity mapping if the proxy receives headers such as user, tenant, trace, or environment.
- Use logging/audit hooks when the user wants observability.
- Ask whether team members connect from different machines. If yes, design the endpoint and host/network assumptions explicitly instead of leaving a laptop-local default.

## Production-Shaped Proxy

Choose this when the user mentions production, staging, governance, approvals, compliance, audit, or multi-tenant use.

- Bind publicly only when a deployment boundary is explicit.
- Configure auth, policies, logging, and Fentaris encrypted secrets before exposure.
- Choose client-to-Fentaris identity separately from upstream auth. Use Fentaris-managed API keys, trusted header identity, or an existing trusted auth boundary for clients; use `oauth()` for OAuth-protected native HTTP/SSE upstreams.
- Persist OAuth tokens with the encrypted local store or an explicit production store; do not rely on in-memory OAuth state across restarts.
- Keep endpoint path and port stable in `fentaris.json`. Keep the default loopback host or configure a supported application-level host option when intentional exposure requires it.
- Do not run or invent deploy commands. State that Fentaris deploy is not available yet and the CLI is expected to add a smoother deploy flow later.

## Existing App Embedding

Choose this when the user already has a TypeScript service and wants Fentaris inside it.

- Add `@fentaris/core` integration in the app's existing structure.
- Prefer `fentaris(...)` as the application boundary.
- Fit startup/shutdown into the app's process model instead of adding a separate unrelated runtime.

## Custom Transport Or Extension

Choose this only when the user needs to connect a non-standard MCP source or custom exposure.

- Prefer built-in transport helpers first.
- For app-owned custom tools/resources/prompts/completions, use `app.local(name)` instead of custom transports or extensions.
- Use extension contracts from `@fentaris/core/extensions` for custom integrations.
- Use advanced low-level proxy/transport APIs only when explicitly requested or required.
