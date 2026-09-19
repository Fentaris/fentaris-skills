# MCP Client Migration

## Migration Invariant

Migration is copy, validate, cut over, then optionally remove. It is never destructive move-first configuration editing.

## Inventory

Use the client's CLI or current official documentation to locate global and workspace MCP configuration. Common filenames and formats change, so do not guess a path and create a second competing config. Inspect only detected/documented config.

For each entry record, without exposing secrets:

- client and config scope;
- stable server name;
- transport: stdio, Streamable HTTP, SSE, or unknown;
- executable plus argument structure, or URL;
- working-directory requirement;
- environment variable names or secret references, not values;
- whether credentials appear user-specific;
- whether the server appears duplicated in another scope/client.

Client-owned skills and agent/persona definitions are inventory context, not MCP servers. Install Fentaris skills into supported coding agents when approved, but leave agent definitions in their client.

## Mapping Into Fentaris

- Map stdio entries to direct `stdio({ command, args, ... })` declarations. Preserve a required working directory. Avoid `sh -lc`, `cmd /c`, or another shell wrapper unless the upstream officially requires one.
- Map native remote servers to `streamableHttp(...)` or `sse(...)` according to the actual transport.
- Detect loopback, private, and link-local upstream or OAuth endpoints. Fentaris rejects them by default. Require explicit trust-boundary approval and prefer a narrow `network.allowedPrivateHosts` allow-list; use broad `allowPrivateNetworkUrls: true` only when its wider SSRF exposure is intentionally accepted.
- Preserve stable, unique names because they become client-visible prefixes such as `github__search_issues`.
- Resolve duplicate names explicitly; never silently merge servers with different commands, URLs, scopes, or credentials.
- Do not migrate an unknown transport until current Fentaris and upstream documentation identify a supported mapping.
- Treat a local app-owned capability differently from an upstream process; use `app.local(...)` only when the capability is implemented in the Fentaris app itself.

## Secret Handling

Configuration may contain literal tokens, environment interpolation, keychain references, or client-managed OAuth state.

- Never include literal values in diffs, logs, prompts, summaries, or command arguments.
- Preserve references when Fentaris can resolve them through its supported secret model.
- Otherwise use `fentaris secrets set <reference> --value-stdin` or inspect `fentaris secrets setup --dry-run --json` and require protected human re-entry. If setup will generate API keys, its apply output contains their raw one-time values; apply only in a verified non-recorded channel or private human terminal.
- Do not copy a client's private OAuth token/cache. Configure `oauth()` and run a new supported Fentaris authorization flow.
- Do not migrate client API keys as upstream credentials or vice versa.

## OAuth Mapping

A remote HTTP/SSE server that uses OAuth should be declared with `oauth()` rather than by copying bearer tokens. Choose per-user authorization only after distinct authenticated Fentaris user identity is configured; unauthenticated callers collapse to a shared OAuth session. Otherwise require explicit approval for shared authorization and explain that callers will share one upstream account. Client credentials also requires explicit trust-boundary approval. Stdio cannot use Fentaris upstream OAuth.

## Client Configuration

After runtime validation, configure one Fentaris endpoint in each selected client using that client's current documented syntax. The ordinary endpoint is:

```text
http://127.0.0.1:4000/mcp
```

Use the actual validated port/path. If client API-key auth is enabled, put `x-fentaris-api-key` in supported secret/header storage and never write it to documentation.

Keep originals until the client successfully initializes against Fentaris and lists expected tools. Then ask whether to retain, disable, or remove them. Store the backup path in the completion report.

## Failure Handling

- If one upstream fails, keep it original and continue with independently valid approved servers.
- If Fentaris runtime validation fails, do not change client configs.
- If a client cannot reach loopback because it is sandboxed or remote, use its approved local networking path; do not bind Fentaris publicly merely as a workaround.
- If config syntax cannot be validated, restore the backup immediately and report the client as not migrated.
