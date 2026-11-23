# Best Practice: Config

A comprehensive guide to Codex CLI's TOML-based configuration system — covering config hierarchy, profiles, sandbox modes, and approval policies.

<table width="100%">
<tr>
<td><a href="../">← Back to Codex CLI Best Practice</a></td>
<td align="right"><img src="../!/codex-jumping.svg" alt="Codex" width="60" /></td>
</tr>
</table>

## Settings Hierarchy

[Config precedence](https://learn.chatgpt.com/docs/config-file/config-basic#configuration-precedence), highest first:

1. CLI flags and `-c key=value` overrides
2. Trusted project `.codex/config.toml` layers, from nearest directory to project root
3. Named user profile selected with `--profile`
4. Base user `$CODEX_HOME/config.toml` (normally `~/.codex/config.toml`)
5. Cloud-managed defaults, system config, then built-in defaults

Managed requirements can constrain the effective settings. Keep shared project defaults in the repo, and personal model and permission choices in user config.

## Profiles

A named config profile is a separate file with ordinary top-level keys. For example, save the following as `$CODEX_HOME/development.config.toml`:

```toml
sandbox_mode = "workspace-write"
approval_policy = "on-request"
```

```bash
codex --profile development
codex exec --profile development "Summarize this diff"
```

Move old `[profiles.<name>]` entries into named files and remove the old top-level `profile` selector. Trusted project config can still override a profile. See [example files](../examples/profiles/) and the [profile reference](https://learn.chatgpt.com/docs/config-file/config-advanced#profiles).

## Sandbox and Approvals

| Sandbox | Filesystem access | Typical use |
|---|---|---|
| `read-only` | Read-only | Review and audit |
| `workspace-write` | Writes within configured workspace roots, with protected paths | Local development |
| `danger-full-access` | Unrestricted by Codex's sandbox | Environments with their own containment |

Network access is controlled separately by the effective sandbox or permission profile. `on-request` lets Codex request approval. `never` disables approval prompts while retaining the sandbox. Granular approval settings are available in TOML. Migrate the retired `untrusted` policy and deprecated `on-failure` setting to a supported policy.

[Named permission profiles](https://learn.chatgpt.com/docs/permissions) are a beta feature. They define filesystem and network boundaries via `[permissions.<name>]` and are selected with `default_permissions` or from `/permissions` when available. Use the permission-profile schema or the legacy sandbox settings as documented for the selected configuration.

### Network Access in Permission Profiles

On macOS, use a narrow Unix socket allowlist when a permitted tool needs a local socket:

```toml
[permissions.development.network]
enabled = true

[permissions.development.network.unix_sockets]
"/tmp/example.sock" = "allow"
```

Select the named permission profile for these settings to take effect. See [Permissions](https://learn.chatgpt.com/docs/permissions) for its filesystem and network schema.

## Shared Project Configuration

- Declare shared integrations under `[mcp_servers.<name>]`. See [MCP](codex-mcp.md).
- Put custom agent files in `.codex/agents/`. Use `[agents]` for shared agent settings. See [Subagents](codex-subagents.md).
- Use `AGENTS.override.md` for temporary instructions in a directory. It takes precedence over that directory's `AGENTS.md`. See [AGENTS.md](codex-agents-md.md).

## One-Off Overrides

```bash
codex --model gpt-6.2-sol -c 'model_reasoning_effort="medium"'
codex exec --sandbox read-only -c 'approval_policy="never"' "Summarize this diff"
```

Pick an available model for your account and provider. Use `/debug-config` to inspect the effective layers, and `--strict-config` to catch unrecognized keys in the CLI. See the [configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference) for supported fields.
