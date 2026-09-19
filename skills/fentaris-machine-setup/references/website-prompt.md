# Official Website Copy-and-Paste Prompt

```text
Set up Fentaris completely on this computer with a zero-hassle experience.

Act autonomously: detect the operating system, inspect the environment, install missing prerequisites, install the official Fentaris CLI and skills, create a working local Fentaris proxy, offer to migrate existing MCP servers from the AI clients already installed, configure the clients I select, and validate the result end to end.

Use only official sources:
- CLI package: @fentaris/cli
- Skills: https://github.com/Fentaris/fentaris-skills
- Documentation: https://fentaris.mintlify.app

If $fentaris-machine-setup is already available, use it. Otherwise follow this prompt as the setup contract. Do not merely explain commands when you have terminal and file access: perform the work. Use an interactive question/dialog tool when available, ask one focused question at a time, and fall back to a short text question when no dialog tool exists.

TARGET STATE

The completed setup must include:
1. a supported Node.js and npm installation (prefer Node 24 LTS on a new machine);
2. the official Fentaris CLI;
3. all official Fentaris skills installed for the AI clients I select;
4. a local Fentaris proxy project, preferably under ~/fentaris-proxy;
5. only the MCP servers I approve migrated behind Fentaris;
6. selected AI clients connected to one validated Fentaris endpoint;
7. credentials and OAuth tokens stored safely;
8. successful static and runtime validation;
9. backups and exact rollback instructions.

SAFETY RULES

- Inventory before changing anything.
- Never expose or repeat passwords, tokens, API keys, client secrets, refresh tokens, or credential values.
- Never place secrets in source code, fentaris.json, prose, logs, shell arguments, or retained agent-terminal output.
- Do not run an API-key generation command whose raw output will be captured in the agent transcript. Use a verified secret-output channel, protected input with --value-stdin, or ask me to run the one generation command in a private terminal.
- Back up every client configuration before editing it.
- Do not remove or disable original MCP entries until Fentaris and that client have both been validated.
- Keep Fentaris bound to 127.0.0.1 unless I explicitly approve network exposure after auth and policy are configured.
- Ask before privileged installation, changing a security boundary, sharing credentials between users, creating autostart, or removing original configuration.
- Do not claim that Codex/Claude/Cursor agent or persona definitions are imported into Fentaris. Fentaris centralizes MCP servers, identity, policy, credentials, and observability; client-specific agents remain in their client.
- Do not claim completion if build, static checks, runtime checks, or selected-client connectivity failed.

PHASE 1 — READ-ONLY INVENTORY

Without modifying the machine:
1. detect OS, architecture, shell, and available package manager;
2. inspect versions or absence of Node.js, npm/npx, pnpm/bun, Git, and Fentaris;
3. detect installed MCP-capable clients such as Codex, Claude Code/Desktop, Cursor, Gemini CLI, OpenCode, and others you can identify reliably;
4. locate their documented global and workspace MCP configuration without scanning unrelated personal data;
5. inventory MCP server names, scope, and transport (stdio, Streamable HTTP, SSE, or unknown);
6. note whether credentials are present without reading them back into the conversation;
7. inventory installed skills and client-owned agent/persona definitions.

Show a short redacted summary only.

PHASE 2 — CONSENT

Ask which detected clients I want to integrate. Offer only clients you actually detected, plus “all detected”, “current client only”, and “Fentaris only”.

Then ask which detected MCP servers to migrate. Show server name, source client, scope, and transport, but no credential values.

If agent/persona definitions exist, explain that they stay in the client. Ask whether to leave them unchanged or install Fentaris skills and configure their client to use the Fentaris endpoint.

Do not ask for information you already discovered. Use safe defaults for non-security choices: a user-owned project directory, loopback host, /mcp, and an available local port.

PHASE 3 — INSTALLATION

After consent:
1. install a supported Node.js only if missing or incompatible; prefer Node 24 LTS for a new machine and use a trusted platform installer;
2. verify node --version and npm --version;
3. install or update the CLI with:
   npm install -g @fentaris/cli
4. verify fentaris --version and fentaris --help;
5. inspect npx skills help/list output and install every official Fentaris skill for each selected supported agent using an explicit target, for example:
   npx skills add Fentaris/fentaris-skills -g -a <agent> --skill '*'
6. do not use --all unless I approve installing into every supported agent;
7. if a client must restart to load skills, continue the setup with official CLI help/docs and report the restart at the end.

PHASE 4 — PROJECT

If no suitable Fentaris project exists:
1. inspect fentaris init --help;
2. create a minimal project with explicit non-interactive options;
3. default to ~/fentaris-proxy, 127.0.0.1, /mcp, and port 4000 or the first free port;
4. use the available package manager and install dependencies;
5. inspect generated files before editing;
6. keep port, endpoint path, entrypoint, and auth directory in fentaris.json; host is not a fentaris.json field, so retain the default loopback binding unless intentional exposure is configured through the supported application option.

If a suitable project already exists, ask whether to reuse it or create a separate one.

PHASE 5 — MCP MIGRATION

For each approved server:
1. preserve a stable unique name, command/arguments or URL, transport, and required working directory;
2. map stdio directly to Fentaris stdio transport without shell wrappers unless officially required;
3. map remote servers to their actual Streamable HTTP or SSE transport;
4. detect loopback/private/link-local upstream and OAuth endpoints, ask before allowing them, prefer a narrow network.allowedPrivateHosts allow-list, and use allowPrivateNetworkUrls: true only if I explicitly accept the broader SSRF exposure;
5. use high-level Fentaris APIs such as app.mcp(...), mcp(...), stdio(...), streamableHttp(...), and sse(...);
6. do not merge duplicates with different commands, URLs, scopes, or credentials without asking;
7. leave unsupported or unknown transports unchanged and report them;
8. move static credentials only through Fentaris encrypted secrets and protected stdin/human input;
9. never copy a client’s OAuth token cache. Configure a new supported Fentaris OAuth flow instead.

PHASE 6 — AUTH

Keep these boundaries separate:
A. clients authenticating to Fentaris;
B. Fentaris authenticating to upstream MCP servers.

For client access, ask whether this is a personal loopback setup or needs Fentaris API keys/users/groups. Use x-fentaris-api-key only when selected, store the key through the client’s supported secret mechanism, and never log it. Because `--generate` prints the raw API key once, only run it through a verified non-recorded secret-output channel; otherwise ask me to generate it in a private terminal or provide it through protected input and `--value-stdin`.

For an OAuth 2.1 upstream:
1. only use oauth() with native Streamable HTTP or SSE, never stdio;
2. use authorization-code + PKCE; choose per-user tokens only after distinct authenticated Fentaris user identity is configured, because unauthenticated callers collapse to the shared OAuth session;
3. for a personal unauthenticated loopback setup, explain the account sharing and ask me to approve shared tokens; use shared tokens or client credentials in any topology only after I approve that security boundary;
4. ensure FENTARIS_AUTH_KEY or an explicit OAuth store provides encrypted persistence;
5. start consent with the current documented fentaris auth login command;
6. let me complete login/consent in the browser;
7. use --print-url for a headless environment;
8. verify with fentaris auth status without exposing tokens.

Do not describe an authorization-code flow as unattended: the human consent step is intentional.

PHASE 7 — VALIDATION GATE

Before changing any client configuration:
1. install project dependencies;
2. run available build/typecheck scripts;
3. run fentaris check --offline --json;
4. run fentaris doctor --json;
5. start the proxy under supervision;
6. run fentaris doctor --runtime --json;
7. list effective tools through Fentaris;
8. verify non-sensitive OAuth/auth status when used;
9. make one safe non-destructive tool call when possible.

Diagnose and fix failures. Keep any failing upstream in its original client configuration and do not cut over a client while the proxy is unhealthy.

PHASE 8 — CLIENT CUTOVER

Only after validation succeeds, for every selected client:
1. create a timestamped backup of its exact config;
2. validate existing syntax;
3. add one MCP server named fentaris pointing to the validated endpoint, normally http://127.0.0.1:4000/mcp;
4. add the API-key header only through supported secret/header storage when enabled;
5. validate syntax again and reload/restart only when required;
6. confirm the client can initialize and list expected proxied tools;
7. then ask whether migrated original MCP entries should remain, be disabled, or be removed while retaining the backup.

If the client is sandboxed or remote and cannot reach loopback, use its approved local networking path. Do not bind Fentaris publicly merely as a workaround.

PHASE 9 — OPTIONAL AUTOSTART

Ask whether Fentaris should start manually, at user login, or through an existing process manager. Do not create a service or scheduled task without consent. If selected, use the least invasive user-level mechanism and provide uninstall instructions.

FINAL REPORT

Return a concise report containing:
- installed versions;
- project directory and endpoint;
- configured clients and required restarts;
- migrated, skipped, and blocked MCP servers;
- non-sensitive client-auth and OAuth status;
- skill targets installed;
- validations and results;
- backup paths;
- start/stop commands;
- rollback steps;
- anything not verified.

If blocked, stop safely and state the exact failed check, what you attempted, and the single permission or input required to continue.
```
