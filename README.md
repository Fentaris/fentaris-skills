# Fentaris Skills

Agent skills for setting up and developing Fentaris projects.

These skills are designed for the open agent skills ecosystem and can be installed with [`npx skills`](https://github.com/vercel-labs/skills). The same repository can target Codex, Claude Code, Cursor, OpenCode, Gemini CLI, and other supported agents.

## Install

Install all Fentaris skills into your current project:

```bash
npx skills add Fentaris/fentaris-skills --skill '*'
```

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

Before the GitHub repo exists, install from a local checkout:

```bash
npx skills add ./fentaris-skills --skill '*'
```

From the parent folder used during local development:

```bash
cd /Users/gabry848/Desktop/panter
npx skills add ./fentaris-skills --skill '*'
```

## Included Skills

### `fentaris-project-setup`

Use this when an agent should introduce Fentaris, ask setup questions, choose the right proxy workflow, create/configure a Fentaris proxy project, validate it, and explain the result.

Good prompt:

```txt
Use $fentaris-project-setup to help me create a Fentaris proxy for my team.
```

### `fentaris-app-development`

Use this when an existing application already uses Fentaris and an agent needs to add or change upstream MCP servers, policy, users/groups, middleware, hooks, approvals, secrets, logging, endpoint configuration, tests, or TypeScript integration.

Good prompt:

```txt
Use $fentaris-app-development to add a GitHub MCP upstream and policy to this app.
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

- OAuth 2.1 setup is not supported by Fentaris yet; the setup skill should guide users toward API keys, trusted headers, or an existing auth boundary when needed.
- Fentaris deploy is not available yet; the setup skill should shape projects so they can move to the future CLI deploy flow cleanly.
- The skills prefer the high-level Fentaris API for normal setup and app changes.
