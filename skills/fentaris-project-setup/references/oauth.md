# Upstream OAuth 2.1

Use this reference when a Streamable HTTP or SSE upstream requires OAuth 2.1. This is the Fentaris client side of the upstream authorization flow; it does not authenticate MCP clients connecting to the Fentaris proxy.

## Supported Shape

Declare `auth: oauth()` on a native `streamableHttp()` or `sse()` upstream. Fentaris performs protected-resource and authorization-server discovery, dynamic client registration when needed, PKCE, token exchange, refresh, and token storage.

```ts
import { fentaris, mcp, oauth, streamableHttp } from "@fentaris/core";

const app = fentaris({
  servers: [
    mcp("linear", {
      transport: streamableHttp({ url: "https://mcp.linear.app/mcp" }),
      auth: oauth(),
    }),
  ],
});

await app.start();
```

Do not attach `oauth()` to stdio or a custom transport that cannot carry an HTTP authorization. Configuration validation rejects unsupported transports.

## Choose The Grant And Token Scope

- `oauth()` — authorization code with discovery, dynamic registration, PKCE, and per-user token mode by default. That isolation only works when callers resolve to distinct authenticated Fentaris users; unauthenticated callers collapse to the shared session.
- `oauth({ clientId, clientSecret, scopes })` — authorization code with a preregistered client. Store a client secret through `credential(...)`, not a literal in source.
- `oauth.clientCredentials(...)` — machine-to-machine access with no human redirect and one shared authorization.
- `tokens: "shared"` — intentionally share the first completed human authorization. Do not select this merely to avoid modeling users.

Ask before choosing a shared identity or client-credentials grant because it changes the security boundary. If client identity is not configured, explain that authorization is effectively shared and obtain explicit approval rather than implying per-user isolation.

## Persistence And Redirects

- With `FENTARIS_AUTH_KEY`, the default store persists encrypted state under `<authDir>/oauth-tokens.enc.json`.
- Without a key or explicit `oauth.store`, state is in memory and is lost on restart. Treat `FENTARIS_CONFIG_OAUTH_STORE_EPHEMERAL` as unfinished setup, not a harmless production default.
- Set `oauth.publicUrl` when the callback must be externally reachable or Fentaris runs behind a reverse proxy. The callback defaults to `/_fentaris/oauth/callback`.
- Never print token, refresh-token, client-secret, or registration-secret values.

## Consent And CLI

For an authorization-code flow, preserve a human consent step:

```bash
fentaris auth login linear --as user:alice
fentaris auth status linear --json
```

Use `--print-url` in a headless environment and `--port` when a preregistered redirect requires a fixed loopback port. `--non-interactive` never opens a browser. Do not claim an authorization-code flow is fully unattended.

Use `fentaris auth logout <mcp> --as user:<id>` to remove one stored authorization. A running proxy observes CLI-written token state without a restart.

## Network Guardrails

OAuth discovery, registration, token, and revocation requests use the upstream transport's network guardrails. Loopback, private, and link-local destinations are rejected by default. Require explicit trust-boundary approval and prefer `network.allowedPrivateHosts` for a narrow allow-list; use `allowPrivateNetworkUrls: true` only when the broader SSRF exposure is intentionally accepted.

## Client Compatibility

When a tool call needs consent, Fentaris can use URL-mode elicitation if the MCP client supports it. Otherwise the call returns `FENTARIS_OAUTH_AUTHORIZATION_REQUIRED` with an authorization URL. Treat that as a required human action, not as a tool failure to bypass.

## Validation

1. Run `fentaris check --offline --json` and `fentaris doctor --json`.
2. Start the proxy.
3. Complete login for the intended user or shared session.
4. Run `fentaris auth status [mcp] --json` without exposing token values.
5. Run `fentaris tools auth status --mcp <mcp> --as <selector> --json` when available.
6. List tools and make one non-destructive call through the intended identity.
