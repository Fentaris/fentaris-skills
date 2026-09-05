# Driving The CLI From An Agent

Use these patterns whenever an agent, script, or CI job runs `fentaris` commands directly, as opposed to a human typing at a terminal.

## General Rules

- Always add `--non-interactive` on commands that support it (`init`, `auth`, `secrets`) so a missing input fails loudly instead of hanging on a prompt.
- Always pass every required value as an explicit flag or argument. Do not rely on interactive/guided flows (omitted `user-id`, omitted `reference`, omitted API-key value) in automation; those exist for humans.
- Prefer `--json` wherever a command supports it (`check`, `doctor`, `secrets list`, `secrets doctor`, `secrets setup`, `tools *`, `edge *`, `auth api-key list`) and parse structured output instead of scraping human-readable text.
- Prefer `--value-stdin` over `--value` for every command that accepts a secret or API-key value (`auth api-key add`, `auth api-key remove`, `secrets set`). Passing secrets as `--value` exposes them in shell history and process listings.
- Prefer `--generate` when the caller does not need to choose the exact key value; capture the printed value exactly once and store it in the user's own secret manager. The CLI stores only a hash, so the value cannot be retrieved again.
- Use `FENTARIS_AUTH_KEY` (not `--key`) to supply the local secrets encryption key in automation; it keeps the key out of process argument lists.
- Never print, log, or persist secret/API-key values yourself, even when the CLI's own output includes a generated value the first time.

## Exit Codes And Failure Semantics

- `fentaris check` and `fentaris doctor` throw/exit nonzero when any check fails, or when `--strict` is set and any check only warned. Treat a nonzero exit as "stop and report", not as a signal to retry blindly.
- `fentaris secrets manifest --check` exits nonzero if `secrets.manifest.json` is missing or stale — useful as a CI gate before merging config changes.
- `fentaris secrets setup` in `--json` or `--non-interactive` mode never prompts and makes no changes while required external values are unavailable; treat "no changes made" in its envelope as "still needs input", not as an error to work around.

## Recommended Command Sequences

### New project (agent-driven)

```bash
fentaris init my-proxy --non-interactive --package-manager pnpm --port 4100 --path /mcp
cd my-proxy
fentaris check --offline --json
```

### Add a user API key (agent-driven)

```bash
# generated (recommended when the agent does not need to choose the value)
fentaris auth api-key add alice --generate

# caller-provided value, never via --value
printf '%s' "$THE_KEY" | fentaris auth api-key add alice --value-stdin

# verify without exposing values
fentaris auth api-key list --user alice --json
```

### Configure required credentials for a project (agent-driven)

```bash
fentaris secrets setup --dry-run --json   # inspect the plan first
fentaris secrets setup --yes --json       # apply once all external values are available
fentaris secrets list --json              # confirm what is now stored
```

### Store one credential directly

```bash
printf '%s' "$GITHUB_TOKEN" | fentaris secrets set github.token --value-stdin
```

### Validate a project end-to-end

```bash
fentaris check --offline --json
fentaris doctor --json
fentaris dev &            # start the proxy
fentaris doctor --runtime --json
```

### Inspect effective tools for an account before wiring policy

```bash
fentaris tools list --as user:alice --json --compact
fentaris tools search "create_issue" --mcp github --json
fentaris tools schema github__create_issue --input --json
```

### Edge operator flows (alpha/preview — confirm the user wants this)

```bash
fentaris edge join https://control.example --name 'Mac Studio' --tag ci --json
fentaris edge list --as user:alice --compact --json
fentaris edge installation status --json
fentaris edge installation approve filesystem --yes --json
```

## When The CLI Prompts Anyway

If a `--non-interactive` run still appears to prompt or hang:

1. Re-check the exact command against `references/commands.md` — a required argument or flag may be missing.
2. Run `fentaris <command> --help` (and `<subcommand> --help`) to confirm the installed CLI version's exact flags; a newer/older CLI may differ from the reference.
3. Do not paper over a missing required value by guessing one. Ask the user for the one input the CLI needs instead of inventing a placeholder secret, name, or ID.
