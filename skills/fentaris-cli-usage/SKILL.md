---
name: fentaris-cli-usage
description: Explain and operate the Fentaris CLI (`fentaris`) itself, command by command. Use when the user asks what a Fentaris CLI command does, which flags to use, how to run it non-interactively, or how commands fit together (init, dev, build, check, doctor, auth, secrets, tools, edge). Use for CLI reference/help questions and for driving the CLI from an agent, not for choosing a project architecture (`fentaris-project-setup`) or for editing TypeScript app code (`fentaris-app-development`).
---

# Fentaris CLI Usage

Use this skill as the CLI reference and operating guide for the `fentaris` command. It complements the other two skills:

- `fentaris-project-setup` decides *what* to build and asks discovery questions.
- `fentaris-app-development` edits *application code* (`fentaris.json`, TypeScript entrypoint).
- `fentaris-cli-usage` (this skill) explains and runs the *CLI itself*: exact commands, flags, exit behavior, and safe automation patterns.

If the user's question is "how do I set up Fentaris" or "how do I add an upstream server", route to those skills instead. If the question is "what does `fentaris doctor --fix` do" or "how do I add an API key non-interactively", stay here.

## Core Workflow

1. Read `references/commands.md` for the full command tree (`init`, `dev`, `build`, `check`, `doctor`, `auth`, `secrets`, `tools`, `edge`) with every documented option, argument, and example.
2. Prefer the installed CLI's own help as the source of truth over memorized command shapes: run `fentaris --help`, `fentaris <command> --help`, or `fentaris <command> <subcommand> --help` before asserting exact flags for an unfamiliar or possibly newer CLI version. Reconcile any difference with `references/commands.md` rather than guessing.
3. When the user wants an explanation, answer directly from `references/commands.md`: what the command does, its required/optional arguments, its important flags, and one runnable example.
4. When the user wants the agent to run CLI commands, read `references/automation.md` first and prefer the non-interactive, explicit-flag form of each command so the run is reproducible and script-safe.
5. Never print secret or API-key values from command output, examples, or logs. Prefer `--value-stdin` over `--value` for anything that accepts a secret, and prefer `--generate` when the caller does not need to choose the value.
6. Route JSON needs to `--json`/`--compact` flags where the command supports them (`check`, `doctor`, `secrets list`, `tools list`, `edge *`) instead of parsing human-readable output.
7. If a command's behavior depends on project state (no `fentaris.json`, missing entrypoint, no local secrets key), explain the likely diagnostic first (`fentaris check`/`fentaris doctor`) instead of guessing why a command failed.
8. Point production/deploy questions to the current limitation: `fentaris` has no deploy command yet; `build` produces a deterministic local artifact only.

## Command Groups At A Glance

- **Project**: `init` (scaffold), `dev` (run in development mode), `build` (deterministic local artifact).
- **Health**: `check` (static project checks), `doctor` (environment + project diagnostics, optional `--fix` and `--runtime`).
- **Identity & secrets**: `auth` / `auth api-key` (local user API keys), `secrets` (credential values and the secrets manifest).
- **Discovery**: `tools` (list/search/inspect effective MCP tools and account auth across configured accounts).
- **Edge** (alpha/preview): `edge` (enroll, run, and operate governed Edge computers/devices).

Every command supports `--help`/`-h`. The root command also supports `--version`/`-v`. `--non-interactive` is a global-style flag honored by commands that would otherwise prompt (`init`, `auth`, `secrets`); use it whenever driving the CLI from an agent or script.

## Safety Defaults

- Never echo secret values, generated API keys, or `FENTARIS_AUTH_KEY` in explanations, logs, or command examples beyond the one-time value the CLI itself prints on generation.
- Default to `--non-interactive` plus explicit flags for any command run by an agent; only fall back to interactive/guided mode when a human is present to answer prompts.
- Do not invent flags. If unsure whether a flag exists on the installed CLI version, check `--help` output or `references/commands.md` before stating it as fact.
- Do not suggest `fentaris deploy`; it does not exist. Redirect deploy questions to `build` plus the user's own hosting/process-manager setup.
- Treat `edge` as alpha/preview: mention that protocol, CLI, and local state formats may still change before recommending it for production reliance.

## Resource Routing

- Read `references/commands.md` for the full command/flag reference and per-command examples.
- Read `references/automation.md` for non-interactive, agent-safe invocation patterns, exit-code handling, and JSON output conventions.
