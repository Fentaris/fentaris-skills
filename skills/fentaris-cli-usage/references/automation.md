# Driving The CLI From An Agent

Use these patterns whenever an agent, script, or CI job runs `fentaris` commands directly, as opposed to a human typing at a terminal.

## General Rules

- Always add `--non-interactive` on commands that support it (`init`, `auth`, `secrets`) so a missing input fails loudly instead of hanging on a prompt.
- Always pass every required value as an explicit flag or argument. Do not rely on interactive/guided flows (omitted `user-id`, omitted `reference`, omitted API-key value) in automation; those exist for humans.
- Prefer `--json` wherever a command supports it (`check`, `doctor`, `secrets list`, `secrets doctor`, `tools *`, `edge *`, `auth api-key list`) and parse structured output instead of scraping human-readable text. `secrets setup --yes --json` is an exception when the plan generates API keys: its JSON contains their one-time raw values and must not run in a retained agent terminal.
- Prefer `--value-stdin` over `--value` for every command that accepts a secret or API-key value (`auth api-key add`, `auth api-key remove`, `secrets set`). Passing secrets as `--value` exposes them in shell history and process listings.
- `auth api-key ... --generate` writes the raw key to stdout once. Use it only in a verified non-recorded secret-output channel or a private human-operated terminal. In an agent terminal with retained output, accept a value through protected input and use `--value-stdin` instead.
- Use `FENTARIS_AUTH_KEY` (not `--key`) to supply the local secrets encryption key in automation; it keeps the key out of process argument lists.
- Never print, log, or persist secret/API-key values yourself.
- Treat OAuth authorization-code login as bounded but human-in-the-loop. `--non-interactive` and `--print-url` prevent automatic browser launch; they do not remove the consent step.

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

### Authorize an OAuth upstream (agent-assisted)

```bash
# Inspect non-sensitive state first
fentaris auth status linear --json

# Interactive desktop: opens the browser and waits for the callback
fentaris auth login linear --as user:alice

# Headless: stdout contains a bare URL before the final result, so do not parse it as one JSON document
fentaris auth login linear --as user:alice --print-url --timeout 300

# Verify without exposing tokens
fentaris auth status linear --json
```

Do not retry a declined or timed-out consent loop blindly. Report the pending human action or provider error. Use `FENTARIS_AUTH_KEY` or a configured OAuth store so the result survives restarts.

### Add a user API key (agent-driven)

```bash
# Use protected input or an existing secret-manager value, never --value
printf '%s' "$THE_KEY" | fentaris auth api-key add alice --value-stdin

# verify without exposing values
fentaris auth api-key list --user alice --json
```

If a new random key is needed, have the human run `fentaris auth api-key add alice --generate` in a private terminal or use a verified secret-output channel that is excluded from the agent transcript.

### Configure required credentials for a project (agent-driven)

```bash
fentaris secrets setup --dry-run --json   # inspect the plan first
fentaris secrets list --json              # confirm what is already stored
```

Inspect the dry-run for generated API-key actions. If the apply step will generate keys, `fentaris secrets setup --yes --json` returns their raw one-time values in `data.generatedApiKeys`; run it only in a verified non-recorded output channel or private human terminal. A recorded agent terminal may apply the plan only when the dry-run proves that no API keys will be generated and every external value is already available.

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
