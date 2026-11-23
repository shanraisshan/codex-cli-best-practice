# Best Practice: Memories

Local Codex memories carry useful context from earlier chats into future work. Keep project rules and shared decisions in `AGENTS.md` or checked-in docs.

## Enable and Control Memories

Local memories are off by default. Enable them in your user config:

```toml
[features]
memories = true

[memories]
use_memories = true
generate_memories = true
disable_on_external_context = true
```

- `use_memories` controls whether new chats receive memory context.
- `generate_memories` controls whether new chats can contribute to future memories.
- `disable_on_external_context` excludes chats that used external context, such as web search, tool search, or MCP. The older `no_memories_if_mcp_or_web_search` is an alias.

Use `/memories` in the CLI to control use and generation for the current chat. The global feature flag determines whether the local memory feature is available. The desktop app also has machine-level settings under **Settings → Personalization**.

## Storage and Updates

Codex generates memories in the background from eligible previous chats. The store belongs to the local Codex home. Existing profiles can use `$CODEX_HOME/memories/`; version 2 uses `$CODEX_HOME/memories_v2/`. Fresh profiles default to v2 when memories are enabled, subject to version and migration settings. During migration both stores may exist.

Optional tuning includes `memories.extract_model`, `memories.consolidation_model`, and `memories.min_rate_limit_remaining_percent`. Leave model overrides unset unless you have verified access and want a specific model for the memory jobs. [Computer History](https://learn.chatgpt.com/docs/customization/computer-history) is a separate macOS desktop app feature.

## Good Habits

- Keep sensitive details out of persistent memories and review generated files before sharing any of them.
- Use checked-in instructions for behavior every contributor should get.
- For throwaway or CI work, leave memories disabled or use an isolated `CODEX_HOME`.

See [local memory controls](https://learn.chatgpt.com/docs/customization/memories#configure-local-memories) and the [configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference).
