---
name: fentaris-project-setup
description: Introduce Fentaris to external users, explain when and why to use it, ask useful discovery questions, choose the right proxy workflow, and create/configure a new Fentaris project when setup is the next step. Use when the user wants an introduction, onboarding, architecture guidance, or initial setup for users/devices/cloud/local usage, MCP upstreams, endpoint/auth/secrets/policies, and validation. If the user already has a Fentaris app and wants to modify it, direct them to fentaris-app-development instead. Do not use for changing the Fentaris framework repository itself.
---

# Fentaris Project Setup

Use this skill to guide an external user from "what is Fentaris and how should I use it?" to a clear recommendation, and only then to a working, validated proxy project when setup is requested.

## Core Workflow

1. Start with a short introduction whenever the user is new, asks broad setup questions, or has not stated a concrete implementation request:
   - Explain that Fentaris centralizes MCP servers behind one controlled endpoint.
   - Explain the value in practical terms: stable MCP client URL, centralized server configuration, users/groups, policy, secrets, logging, and future deploy ergonomics.
   - State that once a project exists, ongoing changes should use the `fentaris-app-development` skill for how to use, modify, debug, and validate Fentaris apps.
   - Avoid long marketing copy; use a compact mental model and then move into the most useful questions.
2. Route the user before implementing:
   - If they need orientation, architecture advice, or first-time setup, continue with this skill.
   - If they already have a Fentaris project and ask to add/change servers, users, policies, approvals, secrets, logging, tests, or TypeScript integration, tell them to use `fentaris-app-development` and switch to that workflow when available.
   - If `fentaris-app-development` is not available in the current agent session, do not inspect Fentaris framework source files or compiled package internals to infer app modification patterns. Tell the user to install all Fentaris skills with `--skill '*'`, or install `fentaris-app-development` explicitly, before continuing with existing-app modification work.
   - If they ask how to use Fentaris after setup, point them to `fentaris-app-development` as the operating/modification skill.
3. Read `references/discovery.md` and ask only the missing high-value questions needed to choose a workflow. Prefer 3-6 concrete questions, grouped by the decision they unlock:
   - Who will use the proxy: one developer, multiple local users, a team, multiple machines, cloud service, or tenant-based app?
   - Where will it run: local laptop, shared workstation, private server, container, cloud runtime, or not decided?
   - Which MCP servers must be connected, and which transport do they use: stdio, Streamable HTTP, SSE/HTTP, or unknown?
   - What auth exists today: API keys, headers from an upstream app, local users, or no auth yet?
   - What controls are needed: users/groups, policies, approvals, logs/audit, encrypted secrets, per-environment routing?
4. Clarify current limitations before designing around them:
   - OAuth 2.1 is not supported yet. Do not design the project around OAuth 2.1 as an available Fentaris feature.
   - Deploy is not available yet. Do not invent deploy commands; keep the project deploy-ready and mention that the CLI is expected to provide a smoother deploy flow later.
5. Read `references/project-types.md` and recommend one setup workflow before creating files. State the recommendation briefly and explain the tradeoff.
6. Ask for confirmation only when the next step creates files, chooses a security boundary, exposes a proxy beyond localhost, or requires secrets. Otherwise proceed with the setup when the user has asked you to build.
7. Use the Fentaris CLI as the source of truth for new projects. Inspect `fentaris init --help` and use explicit CLI options such as package manager, host, port, path, and skip flags when available. Do not hand-build a project or replace generated app code unless the CLI cannot express the requirement.
8. Keep generated projects minimal. Configure the generated files and supported CLI-managed state; do not add custom helper scripts, credential stores, event hooks, `.env` scaffolding, or broad TypeScript rewrites unless the user explicitly asks for that exact capability and the CLI/docs do not provide it.
9. Prefer high-level `@fentaris/core` application builders only for changes the generated project actually requires: `fentaris(...)`, `mcp(...)`, `policy(...)`, `group(...)`, `user(...)`, and transport helpers. Use advanced low-level APIs, `FentarisAuth`, or files under `@fentaris/core/dist/*` only when official docs or generated code require them.
10. Read `references/configuration.md` before editing `fentaris.json`, endpoint, auth, policy, secrets, users, groups, MCP servers, or package scripts.
11. Validate with the most relevant commands available in the project, typically `fentaris check --offline`, `fentaris doctor`, package-manager build/typecheck scripts, and `fentaris doctor --runtime` after starting the proxy.
12. When unsure about current Fentaris behavior, consult CLI help first, then official docs at `https://fentaris.mintlify.app` or local docs if working inside the Fentaris repository. Do not reverse-engineer behavior from compiled `node_modules` unless docs and CLI help are insufficient.
13. Finish with a concise implementation explanation. Read `references/final-explanation.md` for the expected shape.

## Agent-Friendly CLI Use

Prefer explicit, automation-safe inputs. Use `--non-interactive` for agent-driven Fentaris CLI commands and pass required options explicitly. If the CLI asks for interactive input, inspect the command help for the equivalent option; ask the user only when no explicit option exists or the prompt is a real product/security choice.

For secrets, use Fentaris encrypted secret management through the CLI with `--non-interactive`. Do not use environment variables as the default secret storage path. If the exact command shape is uncertain for the installed CLI version, inspect `fentaris secrets --help` or the current Fentaris docs, then use the documented non-interactive command. Never print secret values in logs, code, docs, or final responses.

For non-secret runtime configuration, use `fentaris.json` as the source of truth. Put host, port, endpoint path, runtime entrypoint, and local auth directory in `fentaris.json`; do not default these values to environment variables in generated setup guidance.

For per-agent identity or statistics, use supported Fentaris users/groups/identity configuration or CLI commands. If the installed CLI/API does not expose a supported way to register agents yet, explain the gap and leave the project ready for it; do not invent a separate `scripts/add-agent.ts`, custom credentials file, or manual encryption workflow.

## Safety Defaults

- Keep local development bound to `127.0.0.1` unless the user intentionally exposes it.
- Keep host, port, and the client endpoint path in `fentaris.json`; keep the path stable, usually `/mcp`.
- Add auth and policy before recommending exposure outside local development.
- Treat OAuth 2.1 as unsupported for now; suggest API-key/header-based identity or a trusted upstream auth boundary when appropriate.
- Do not print secret values. Store secrets with Fentaris encrypted secrets via the CLI using `--non-interactive`; do not default to `.env` files or raw environment variables for secret values.
- Avoid deep imports from `@fentaris/core/dist/*` or source-layout paths.
- Avoid shell-wrapped upstream commands such as `sh -lc` unless the upstream MCP server officially requires them. Prefer direct command/args/env declarations supported by Fentaris and the upstream server docs.

## Resource Routing

- Read `references/discovery.md` before asking onboarding/setup questions.
- Read `references/project-types.md` to choose between local demo, team proxy, production-shaped proxy, existing-app embedding, or custom transport work.
- Read `references/configuration.md` for concrete setup and validation guidance.
- Read `references/final-explanation.md` before the final response for setup tasks.
