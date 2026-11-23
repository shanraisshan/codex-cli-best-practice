# Best Practice: MCP (Model Context Protocol)

Use MCP to connect Codex CLI to tools and resources. Configure servers in `config.toml` or use `codex mcp`.

## Configure a Server

```toml
[mcp_servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "."]

[mcp_servers.internal_docs]
url = "https://docs.example.com/mcp"
bearer_token_env_var = "DOCS_MCP_TOKEN"
enabled_tools = ["search"]
```

For stdio, `env_vars` forwards named environment variables from the parent process. For Streamable HTTP, use `bearer_token_env_var` or authenticate using `codex mcp login <name>` when the server supports OAuth. Keep actual secrets out of checked-in TOML.

## Manage Access

- Use `codex mcp list` or `/mcp` to inspect configured servers. Use `codex mcp add --help` for supported connection flags.
- Limit exposed tools with `enabled_tools` or `disabled_tools` and set tool-specific approval behavior when needed.
- Configure additional connections in a custom agent's TOML when only that role needs them.
- Use `supports_parallel_tool_calls = true` for a server whose tools can safely run concurrently. Leave it off when calls can conflict.
- Review the server and its requested access before enabling it. Sandboxing of Codex shell commands is separate from permissions granted to a server or app.

### Per-Tool Approval

For servers that mix access levels, request approval on a write tool:

```toml
[mcp_servers.internal_docs.tools.publish]
approval_mode = "prompt"
```

ChatGPT apps are configured separately under `[apps.<id>]`. Their tool availability and approval settings depend on the account and workspace. MCP servers can also expose resources and request structured user input when the client and server support it.

## Migrate From Codex as an MCP Server

The `codex mcp-server` command and standalone binary were removed. Use the experimental [Codex App Server](https://learn.chatgpt.com/docs/app-server) for a client that needs streamed Codex events, approvals, and session history. It uses its own JSON-RPC protocol. Outbound MCP connections still use `codex mcp`.

See the [MCP configuration guide](https://learn.chatgpt.com/docs/extend/mcp) and [server removal notice](https://learn.chatgpt.com/docs/mcp-server).
