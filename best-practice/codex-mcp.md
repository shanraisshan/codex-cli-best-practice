# Best Practice: MCP (Model Context Protocol)

Codex CLI uses `[mcp_servers.<name>]` in `.codex/config.toml` for MCP
integrations, and it can also run as an MCP server via `codex mcp-server`.

## MCP Server Configuration

```toml
[mcp_servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "."]

[mcp_servers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_TOKEN = "$GITHUB_TOKEN" }
```

## Agent-Scoped MCP Servers

Keep MCP access narrow by attaching servers to specific agents:

```toml
[agents.data-analyst]
description = "Works with repository and database context"
config_file = "agents/data-analyst.toml"
```

```toml
# .codex/agents/data-analyst.toml
model = "o4-mini"
mcp_servers = ["filesystem", "github"]
```

## Codex as an MCP Server

```bash
codex mcp-server
```

Consumer example:

```json
{
  "mcpServers": {
    "codex": {
      "command": "codex",
      "args": ["mcp-server"]
    }
  }
}
```

## Tool Namespacing

Codex automatically namespaces MCP tools as `mcp__<server>__<tool>`, using the
table key under `[mcp_servers.<server>]`. There is no separate `namespace = "..."`
field — just pick a stable table key:

```toml
[mcp_servers.docs]
command = "docs-server"

[mcp_servers.docs.tools.search]
approval_mode = "prompt"
```

The model sees `mcp__docs__search`. Collisions across servers are impossible
because the server key is always part of the tool name.

## Parallel Tool Calls (v0.121.0+)

MCP tools are serialized by default. Opt a specific server into parallel
execution when its tools are safe to run concurrently:

```toml
[mcp_servers.docs]
command = "docs-server"
supports_parallel_tool_calls = true
```

This is per-server — there is no global switch. Leave it off for any server
that mutates shared state.

## MCP Apps (v0.119.0+)

Codex supports richer MCP capabilities beyond plain tool calls:

| Capability | How it surfaces |
|---|---|
| Resource reads | Model uses built-in `list_mcp_resources` / `read_mcp_resource` tools |
| Tool-call metadata | Codex sends `_meta` with requests; reads `_meta` from results |
| Custom-server tool search | `/mcp` tool search extends to your own servers |
| Server-driven elicitations | TUI renders a form from a JSON schema the server sends |
| File-parameter uploads | Server declares `_meta: { "openai/fileParams": ["file", ...] }`; Codex rewrites the param schema to accept absolute local paths |

Approval policy `Never` auto-rejects elicitations. `DangerFullAccess` auto-
approves them.

### `codex_apps` Built-In Server

When the user is ChatGPT-authed and `apps_enabled = true`, Codex registers a
reserved MCP server named `codex_apps` backed by the ChatGPT backend. It is
configured under `[apps.<id>]` rather than `[mcp_servers.*]`:

```toml
[apps.my_app]
enabled                     = true
default_tools_approval_mode = "prompt"   # auto | prompt | approve
default_tools_enabled       = true
destructive_enabled         = false
open_world_enabled          = false

[apps.my_app.tools.send_email]
approval_mode = "prompt"
enabled       = true
```

## Sandbox-State Metadata (v0.121.0+)

An MCP server can advertise the experimental capability
`codex/sandbox-state-meta` in its `initialize` result. When set, Codex
augments every `tools/call` `_meta` with:

```json
{
  "codex/sandbox-state-meta": {
    "sandboxPolicy": { "...": "..." },
    "codexLinuxSandboxExe": "/path/to/codex-linux-sandbox",
    "sandboxCwd": "/workspace",
    "useLegacyLandlock": false
  }
}
```

This lets trusted MCP servers spawn child processes under the same sandbox as
the Codex session. There is no Codex-side config switch — only the server
decides whether to opt in.

## Security Guidance

1. Use `$ENV_VAR` references for secrets
2. Scope MCP servers per agent instead of making every server globally available
3. Prefer `workspace-write` or `read-only` when a server does not need network
4. Switch to `danger-full-access` only when the MCP workflow truly requires network access

## Anti-Patterns

- Documenting the retired MCP table naming from older Codex releases
- Hardcoding tokens in config
- Giving every agent the same MCP surface area
- Using the retired one-flag MCP server syntax instead of `codex mcp-server`
