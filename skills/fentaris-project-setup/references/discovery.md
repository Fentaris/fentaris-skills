# Discovery

Use this reference before creating a new Fentaris project or when introducing Fentaris. The goal is to help the user understand the proxy model, ask useful questions, and choose a setup workflow, not to interrogate them.

## Route First

Before asking setup questions, decide which help the user needs:

- Introduction or architecture guidance: use this setup skill.
- Brand-new proxy project: use this setup skill, then proceed to project creation after discovery.
- Existing Fentaris app changes: tell the user to use `fentaris-app-development`, the skill for using, modifying, debugging, and validating Fentaris apps.
- Fentaris framework repository changes: do not use this skill.

## Short Introduction

When the user is new to Fentaris, explain this mental model:

- MCP clients usually need to know every server they connect to.
- Fentaris puts one controlled proxy endpoint in front of those MCP servers.
- The client connects to one stable URL, while Fentaris handles upstream server configuration, app-owned local MCP capabilities, naming, users/groups, API keys, policy, secrets, logging, and runtime checks.

Keep the explanation grounded in their case and end with questions that move the decision forward. For example:

- Solo local setup: "You get one local endpoint and can add/remove MCP servers without reconfiguring every client."
- Team setup: "The team shares one governed MCP entrypoint instead of each machine having a different MCP setup."
- Cloud or internal service: "The app can apply identity, policy, and audit rules at the edge before MCP tools run."

## Discovery Questions

Ask only the unanswered questions needed for the next decision. Prefer 1-4 questions when the user already gave project name, upstreams, auth intent, or runtime scope; use 3-6 only for broad architecture requests. If enough information is present to create a local project safely, proceed instead of asking for perfect detail.

### Orientation

- Are you trying to understand Fentaris, set up a first proxy, or change an existing Fentaris app?
- What is the first MCP client or app that should connect to Fentaris?
- What pain are you trying to remove: repeated client configuration, shared team setup, policy/governance, encrypted secret handling, logging, or multi-user routing?
- Do you want a quick local prototype or a project shaped for a team/server later?

### Users And Access

- Is this for one developer, a small team, or users of an application?
- Should different users or groups see different tools?
- Are users known locally, passed from an existing app, or not defined yet?
- Should users authenticate with Fentaris-managed API keys in the `x-fentaris-api-key` header?
- Is there an existing auth boundary, such as API keys, headers from your backend, or a gateway?

### Machines And Runtime

- Will the proxy run on the same machine as the MCP client, on a shared machine, or in cloud/server infrastructure?
- Do users need to connect from different computers?
- Should it be local-only for now, or shaped for a future shared deployment?
- Which package manager and TypeScript style does the project use?

### MCP Servers

- Which MCP servers should be connected first?
- For each server, is it stdio, Streamable HTTP, SSE/HTTP, or unknown?
- Which servers require credentials that should be stored as Fentaris encrypted secrets?
- Does Fentaris need to expose custom/local MCP tools, resources, prompts, or completions implemented in the app itself?
- Which tools should be allowed initially, and which should be blocked?

### Governance

- Are policies needed now, or is this an initial local prototype?
- Are approvals needed for sensitive tools?
- Should calls be logged for debugging, audit, or both?
- Are there staging/production differences to plan for?

## Current Limits To Surface

- OAuth 2.1 is not supported yet. Do not promise OAuth 2.1 setup. If the user asks for OAuth, explain the current limit and suggest a temporary architecture with API keys, trusted upstream identity headers, or an external auth boundary.
- Fentaris deploy is not available yet. Do not run deploy commands. Shape the project so it can move cleanly to a future CLI deploy flow.

## Recommendation Shape

After discovery, recommend one workflow:

```md
I would set this up as a <workflow name>.

Reason:
- <why this fits users/runtime>
- <why this fits MCP servers/auth>
- <what we keep flexible for later>

Initial project shape:
- Endpoint: <local/shared/cloud-shaped>
- Users/auth: <local users/Fentaris API keys/header identity/none yet>
- MCP servers: <list>
- Local custom capabilities: <none/tools/resources/prompts/completions>
- Policies: <starting policy>
- Validation: <checks to run>
```

Then ask for confirmation only if a decision is risky or irreversible. Otherwise proceed with the setup.

For a solo local proxy with known upstream names and auth requested, the usual recommendation is:

- Local Developer Proxy
- `127.0.0.1` host, `/mcp` path, generated or user-selected port
- Fentaris-managed auth/identity if available
- Fentaris encrypted secrets for upstream credentials
- `fentaris auth api-key` for local user API keys when user auth is needed
- Broad development policy only when explicitly labeled temporary
