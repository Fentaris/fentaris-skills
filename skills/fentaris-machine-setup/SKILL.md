---
name: fentaris-machine-setup
description: Install and bootstrap Fentaris on a computer, install the official Fentaris skills for selected AI clients, inventory existing MCP client configurations, migrate approved MCP servers into a local Fentaris proxy, configure selected clients to use it, and validate with backups and rollback. Use for new-computer, zero-effort, machine-wide, Codex/Claude/Cursor/Gemini/OpenCode, or existing-MCP migration requests. Do not use for ordinary changes inside an existing Fentaris app.
---

# Fentaris Machine Setup

Use this skill for whole-machine onboarding. The result is a validated Fentaris project plus selected AI clients connected to one stable endpoint; it is not merely a list of commands for the user to run.

## Ownership And Routing

- This skill owns prerequisite installation, client inventory, migration planning, backups, client reconfiguration, runtime validation, and rollback.
- This skill remains the orchestrator for the entire machine setup. Apply `fentaris-project-setup` guidance only to the bounded project-shape/app-generation subtask; do not hand the overall request back and forth between skills.
- Apply `fentaris-app-development` guidance to the bounded TypeScript changes that represent approved upstreams, auth, and policy.
- Apply `fentaris-cli-usage` guidance for exact CLI flags and current help.
- Agent/persona definitions remain owned by their AI client. Fentaris centralizes MCP servers, identity, policy, credentials, and observability; it does not import client-specific agent definitions as Fentaris objects.

If another referenced skill is unavailable, use official Fentaris CLI help and documentation rather than framework internals or guessed commands.

## Core Workflow

1. Read `references/workflow.md` before changing the machine.
2. Inventory silently first: OS, architecture, shell, Node/npm, package managers, Git, Fentaris CLI, supported AI clients, their MCP configs, installed skills, and agent/persona definitions. Redact every secret value.
3. Present only detected clients and MCP server names/types. Ask one focused question at a time, using an interactive dialog tool when available:
   - which detected clients to integrate;
   - which detected MCP servers to migrate;
   - whether to install Fentaris skills for those clients;
   - whether upstream OAuth, client identity, or autostart decisions are actually needed.
4. Install only missing prerequisites and the official `@fentaris/cli`. Inspect `--help` before relying on exact flags. Ask before a privileged/system-wide installation when the platform requires it.
5. Install this repository's complete skill set for each selected supported agent with an explicit `npx skills` target. Do not use `--all` without consent.
6. Create or select a Fentaris project. Default new local setups to a user-owned directory, `127.0.0.1`, path `/mcp`, and a free local port. Use `fentaris init`; do not hand-build a scaffold the CLI can generate.
7. Read `references/migration.md`, back up every client config before editing, and migrate only approved MCP entries. Keep originals active until the proxy passes runtime validation.
8. Keep credentials out of source, config, arguments, logs, and final output. Use Fentaris encrypted secrets, stdin, client secret storage, and human re-entry when a value cannot be migrated safely.
9. Distinguish client-to-Fentaris auth from upstream OAuth 2.1. Use `oauth()` only for native Streamable HTTP/SSE upstreams. Human consent is required for authorization-code flows; use `fentaris auth login` or `--print-url` rather than pretending the flow is unattended.
10. Validate before cutover: dependency install, build/typecheck when present, `fentaris check --offline --json`, `fentaris doctor --json`, controlled proxy start, `fentaris doctor --runtime --json`, tool listing, and one safe non-destructive call when possible.
11. Only after validation, add one Fentaris endpoint to selected clients. Verify the client connection, then ask whether to retain, disable, or remove migrated originals. Preserve backups regardless.
12. Ask before installing autostart or a persistent service. Prefer the least invasive user-level mechanism and provide removal instructions.
13. Finish with versions, project path, endpoint, configured clients, migrated/skipped MCPs, non-sensitive auth status, installed skills, validation results, backup paths, start/stop commands, and rollback.

## Non-Negotiable Safety

- Never display or copy plaintext secrets into source, prose, logs, or command arguments.
- Never overwrite a client config without a timestamped backup and syntax-preserving edit.
- Never remove original MCP entries before proxy and client validation.
- Never expose beyond loopback without explicit intent plus auth and policy.
- Never silently convert a per-user credential into a shared credential. Per-user OAuth requires a distinct authenticated Fentaris user identity; unauthenticated callers collapse to the shared OAuth session.
- Never run `fentaris auth api-key ... --generate` through a recorded agent terminal unless the environment provides a verified secret-output channel. Otherwise have the human generate it in a private terminal or provide a value through protected input and `--value-stdin`.
- Never claim agent/persona definitions were imported into Fentaris.
- Never claim completion when static checks, runtime checks, or the selected client connection failed.
- Stop with the exact blocker and the single permission/input needed when no safe automated path remains.

## Resource Routing

- Read `references/workflow.md` for prerequisite installation, questions, validation, cutover, and rollback.
- Read `references/migration.md` before inspecting or changing AI-client MCP configuration.
- `references/website-prompt.md` is the English copy-and-paste prompt for the official website. Use it as the behavioral contract when a user arrives through that prompt.
