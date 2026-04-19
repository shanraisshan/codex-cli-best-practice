# Best Practice: Config

A comprehensive guide to Codex CLI's TOML-based configuration system — covering config hierarchy, profiles, sandbox modes, and approval policies.

<table width="100%">
<tr>
<td><a href="../">← Back to Codex CLI Best Practice</a></td>
<td align="right"><img src="../!/codex-jumping.svg" alt="Codex" width="60" /></td>
</tr>
</table>

## Settings Hierarchy

Settings apply in order of precedence (highest to lowest):

| Priority | Location | Scope | Purpose |
|----------|----------|-------|---------|
| 1 | CLI flags / `-c key=value` | Invocation | One-off overrides for a single run |
| 2 | `.codex/config.toml` | Project | Team-shared defaults, profiles, MCP, agents |
| 3 | `~/.codex/config.toml` | Global | Personal defaults across projects |

## Core Configuration

```toml
# .codex/config.toml
model = "o4-mini"
sandbox_mode = "workspace-write"
approval_policy = "on-request"
```

## Profiles

Named presets under `[profiles.<name>]` — switch with `codex --profile <name>`:

```toml
[profiles.conservative]
sandbox_mode = "read-only"
approval_policy = "untrusted"

[profiles.development]
sandbox_mode = "workspace-write"
approval_policy = "on-request"

[profiles.ci]
model = "o4-mini"
sandbox_mode = "read-only"
approval_policy = "never"

[profiles.trusted]
sandbox_mode = "danger-full-access"
approval_policy = "never"
```

Set a default profile with `profile = "conservative"` at the top level.

## Sandbox Modes

| Mode | File Access | Network | Best for |
|---|---|---|---|
| `read-only` | Read-only project access | Blocked | Reviews, audits, CI analysis |
| `workspace-write` | Read/write inside the workspace | Blocked | Local development and doc/code edits |
| `danger-full-access` | Unrestricted filesystem access | Allowed | Fully trusted automation that needs network or installs |

## Approval Policies

| Policy | Behavior | Best for |
|---|---|---|
| `untrusted` | Auto-runs only trusted read-style commands; asks for the rest | New repos, audits, reviews |
| `on-request` | Model decides when it should ask | Everyday development |
| `never` | Never asks; failures come straight back to the model | Non-interactive runs and tightly controlled automation |

## Memories (v0.119.0+)

Enable the cross-session memory pipeline:

```toml
[features]
memories = true

[memories]
use_memories      = true
generate_memories = true
```

Full reference: [`codex-memory.md`](codex-memory.md).

## Unix Socket Allowlists (macOS, v0.121.0+)

Under Seatbelt, Unix sockets are denied by default. Allowlist specific paths
per-permission-profile:

```toml
[permissions.development.network]
enabled = true

[permissions.development.network.unix_sockets]
"/tmp/example.sock"        = "allow"
"/var/run/myservice.sock"  = "allow"
```

Paths are matched as subpath prefixes, so any socket created under an allowed
directory is covered. Use `dangerously_allow_all_unix_sockets = true` only in
throwaway sandboxes.

## Secure Devcontainer (v0.121.0+)

Codex ships a hardened VS Code Dev Container profile for customer environments.
It is a standalone `.devcontainer/devcontainer.secure.json` + `Dockerfile.secure`
— not a `[profiles.*]` entry in `config.toml`:

```bash
devcontainer up --workspace-folder . \
  --config .devcontainer/devcontainer.secure.json
```

The image installs `bubblewrap` setuid, sets `CODEX_ENABLE_FIREWALL=1`, and
restricts egress via `OPENAI_ALLOWED_DOMAINS`. Use this when running Codex
against untrusted code.

## Override

Use `AGENTS.override.md` for personal instruction overrides — loaded before `AGENTS.md`, not committed to git.

## MCP Servers

Declare shared integrations in the same config file:

```toml
[mcp_servers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_TOKEN = "$GITHUB_TOKEN" }
```

## Agents

Register agents under `[agents.<name>]` and optionally point them at dedicated role files:

```toml
[agents.backend-dev]
description = "Handles backend implementation tasks"
config_file = "agents/backend-dev.toml"
```

## One-Off Overrides

```bash
codex -c model=\"o3\" -c approval_policy=\"never\" exec "summarize this diff"
```

## Anti-Patterns

- Using `danger-full-access` for ordinary editing tasks
- Treating `never` as a general-purpose local default
- Using `danger-full-access` and `never` together without a real containment boundary
- Hardcoding secrets instead of using `$ENV_VAR` expansion
- Mixing unrelated concerns into one profile instead of creating focused profiles
