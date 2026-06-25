# API Guidance

## Default API Tier

Prefer the high-level application builders from `@fentaris/core`:

- `fentaris(...)` for the app boundary.
- `app.mcp(...)` or `mcp(...)` for upstream declarations.
- `app.local(...)` for custom local MCP servers and custom methods/capabilities implemented inside the Fentaris app.
- `policy(...)`, `group(...)`, and `user(...)` for authorization.
- Built-in transport helpers for stdio and HTTP-style upstreams.

This style keeps servers, users, groups, policies, endpoint settings, logging, and hooks readable in one app model.

## Custom Local MCP Capabilities

Use `app.local(name)` when the app needs to expose custom behavior as an MCP server from inside Fentaris. The local namespace becomes a normal Fentaris MCP server for policy, middleware, events, logging, and client-visible names.

```ts
import { Policy, fentaris, streamableHttp } from "@fentaris/core";

const app = fentaris({
  policy: new Policy({ name: "workspace" })
    .mcp("workspace")
    .allow("status")
    .mcp("docs")
    .allow("*"),
});

app.mcp("docs", {
  transport: streamableHttp({ url: "https://docs.example.com/mcp" }),
});

app.local("workspace")
  .tool("status", { inputSchema: { type: "object" } }, (ctx) => ({
    content: [{ type: "text", text: `ok:${ctx.auth.userId ?? "anonymous"}` }],
  }))
  .resource("config://current", { name: "Current config" }, (ctx, params) => ({
    contents: [{ uri: params.uri, text: JSON.stringify({ user: ctx.auth.userId }) }],
  }))
  .prompt("review_pr", { arguments: [{ name: "diff" }] }, (_ctx, params) => ({
    messages: [{ role: "user", content: { type: "text", text: String(params.arguments?.diff ?? "") } }],
  }));

await app.start();
```

Rules for local capabilities:

- Use stable local namespace names. They are server names and affect client-visible names such as `workspace__status`.
- Do not reuse a name for both `app.local(name)` and an upstream `app.mcp(name, ...)`.
- Local handlers receive `ProxyContext` and the MCP operation params; return normal MCP SDK results.
- Supported local declarations are `tool`, exact `resource`, `resourceTemplate`, `prompt`, and `completion`.
- Policy should explicitly allow the local tools/capabilities just like an upstream MCP server.
- Prefer `app.local(...)` over custom transports or plugin experiments when the goal is to expose app-owned custom methods.

## Import Rules

- Use `@fentaris/core` for application code.
- Use `@fentaris/core/extensions` only when implementing custom extension contracts or avoiding name collisions with top-level helpers.
- Avoid `@fentaris/core/dist/*` and source-layout paths.

## Advanced Wiring

Fentaris also exposes lower-level proxy and transport construction for explicit wiring. Do not choose it by default. Use it only when:

- The user explicitly requests direct proxy/transport wiring.
- The app already uses that style and consistency matters.
- A custom downstream exposure or transport requirement cannot be represented through high-level declarations or `app.local(...)`.

When using advanced wiring, check the official docs first because this surface is more fragile during alpha.

## Plugins

Experimental plugin contracts exist, but they are not a production plugin runtime. Do not build a user solution around plugin loading unless the user is explicitly exploring experimental contracts.
