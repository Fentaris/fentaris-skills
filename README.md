# Fentaris Skills

Agent skills for installing Fentaris, migrating existing MCP client configuration, and creating or developing Fentaris projects.

These skills are designed for the open agent skills ecosystem and can be installed with [`npx skills`](https://github.com/vercel-labs/skills). The same repository can target Codex, Claude Code, Cursor, OpenCode, Gemini CLI, and other supported agents.

## Install

Install all Fentaris skills into your current project. This is the recommended path because the setup skill routes existing app work to the app-development skill:

```bash
npx skills add Fentaris/fentaris-skills --skill '*'
```

Avoid installing only `fentaris-project-setup` unless you only want project-level onboarding and brand-new project creation. Whole-machine bootstrap and MCP client migration need `fentaris-machine-setup`, while existing Fentaris app changes need `fentaris-app-development`.

Install globally for Codex:

```bash
npx skills add Fentaris/fentaris-skills -g -a codex --skill '*'
```

Install globally for Claude Code:

```bash
npx skills add Fentaris/fentaris-skills -g -a claude-code --skill '*'
```

Install globally for every supported agent:

```bash
npx skills add Fentaris/fentaris-skills -g --all
```

To test a local checkout before publishing changes, run this from its parent directory:

```bash
npx skills add ./fentaris-skills --skill '*'
```

## Included Skills

### `fentaris-machine-setup`

Use this for a new-computer or whole-machine setup: install prerequisites and the CLI, install these skills for selected AI clients, inventory existing MCP configuration, create a local proxy, migrate approved servers safely, configure selected clients, and validate the result with backups and rollback instructions.

Good prompt:

```txt
Use $fentaris-machine-setup to install Fentaris on this computer, offer to migrate my existing MCP servers, and configure my selected AI clients.
```

The copy-and-paste prompt intended for the Fentaris website is available at [`skills/fentaris-machine-setup/references/website-prompt.md`](./skills/fentaris-machine-setup/references/website-prompt.md).

### `fentaris-project-setup`

Use this when an agent should introduce Fentaris, ask useful setup questions, route existing app work to `fentaris-app-development`, choose the right proxy workflow, create/configure a Fentaris proxy project, add supported user API keys, validate it, and explain the result.

Good prompt:

```txt
Use $fentaris-project-setup to help me create a Fentaris proxy for my team.
```

Good intro prompt:

```txt
Use $fentaris-project-setup to explain how Fentaris fits my MCP setup and what questions I should answer before creating a project.
```

### `fentaris-app-development`

Use this when an existing application already uses Fentaris and an agent needs to add or change upstream MCP servers, custom local MCP capabilities, policy, users/groups, API keys, middleware, hooks, approvals, secrets, logging, endpoint configuration, tests, or TypeScript integration.

Good prompt:

```txt
Use $fentaris-app-development to add a GitHub MCP upstream and policy to this app.
```

### `fentaris-cli-usage`

Use this when an agent needs to explain or run a `fentaris` CLI command itself: what a command/subcommand does, which flags it accepts, how to run it non-interactively, and how to read its JSON output. Covers `init`, `dev`, `build`, `check`, `doctor`, upstream OAuth and local API-key commands under `auth`, `secrets`, `tools`, and `edge`. It does not choose project architecture (`fentaris-project-setup`) or edit application TypeScript (`fentaris-app-development`).

Good prompt:

```txt
Use $fentaris-cli-usage to explain fentaris doctor --fix and run it non-interactively.
```

## List Skills

```bash
npx skills add Fentaris/fentaris-skills --list
```

For a local checkout:

```bash
npx skills add ./fentaris-skills --list
```

## Notes

- Fentaris supports OAuth 2.1 for native Streamable HTTP and SSE upstreams through `oauth()`, including PKCE, refresh, per-user authorization, and encrypted token storage. This is upstream authentication; client access to the Fentaris proxy remains a separate identity/auth decision.
- Fentaris deploy is not available yet; the setup skills should shape projects so they can move to the future CLI deploy flow cleanly.
- The skills prefer the high-level Fentaris API for normal setup and app changes, including `app.mcp(...)` for upstreams and `app.local(...)` for app-owned custom MCP capabilities.
- Fentaris CLI 1.1.0 and newer can manage local user API keys with `fentaris auth api-key`; the skills should use that instead of custom registration scripts.
