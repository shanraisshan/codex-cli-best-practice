# Best Practice: Memories

Codex CLI v0.119.0+ introduced **memories** — a per-user context system that
learns from your threads and injects relevant notes into future sessions.
Memories are scoped to your `$CODEX_HOME` (not per-project), opt-in per thread,
and fully resettable.

<table width="100%">
<tr>
<td><a href="../">← Back to Codex CLI Best Practice</a></td>
<td align="right"><img src="../!/codex-jumping.svg" alt="Codex" width="60" /></td>
</tr>
</table>

## Feature Flag

Memories are off by default. Enable in `config.toml`:

```toml
[features]
memories = true
```

## Slash Command

In the TUI, open the memories settings view with `/memories`. Three controls:

| Item | Effect |
|---|---|
| **Use memories** | Inject existing memories into new threads (applied at next thread) |
| **Generate memories** | Include current + future threads as memory-generation input |
| **Reset all memories** | Clear local memory files and summaries. Existing threads stay intact |

Reset asks for confirmation and wipes both `$CODEX_HOME/memories/` and
`$CODEX_HOME/memories_extensions/` — it does NOT touch the threads themselves.

A hidden CLI equivalent also exists for scripted environments:

```bash
codex debug clear-memories
```

## Config Keys

Full `[memories]` table (all optional, shown with defaults):

```toml
[memories]
use_memories                       = true     # inject into future sessions
generate_memories                  = true     # record threads as generation input
consolidation_model                = "gpt-5.4"        # Phase 2 model
extract_model                      = "gpt-5.4-mini"   # Phase 1 model
max_raw_memories_for_consolidation = 256      # cap 4096
max_rollout_age_days               = 30       # clamp 0..=90
max_rollouts_per_startup           = 16       # cap 128
max_unused_days                    = 30       # drives memory-extension cleanup (0..=365)
min_rollout_idle_hours             = 6        # clamp 1..=48
no_memories_if_mcp_or_web_search   = false    # privacy guard
```

The pair `use_memories` + `generate_memories` replaces a single mode enum — a
"use-only" client sets `generate_memories = false` and still benefits from
previously-generated memories.

## Per-Thread Override

Each thread carries a `memory_mode` column in the state DB (`"enabled"` or
`"disabled"`). Toggled from the TUI settings view or programmatically via the
app-server RPC `thread/memoryMode/set`.

## Memory Extensions

An extension is a plug-in that contributes both persistent instructions and
transient per-run evidence. On disk:

```
$CODEX_HOME/memories_extensions/
└── <extension_name>/
    ├── instructions.md              # always loaded if present
    └── resources/
        └── 2026-04-18T14-32-05-notes.md   # per-run artifact, auto-pruned
```

Cleanup runs during Phase 2 consolidation: resource files older than 7 days are
deleted automatically. A full reset wipes the entire `memories_extensions/`
tree.

## Scope Summary

| Scope | Location | Notes |
|---|---|---|
| Per-thread | state DB `threads.memory_mode` | `/memories` toggles or RPC |
| Per-user (global) | `$CODEX_HOME/memories/` | Consolidated raw + summaries |
| Project | *(not supported)* | Memories do not scope to a project directory |

Memory pipeline only runs for root sessions that are: non-ephemeral, feature-
enabled, not a sub-agent, and with state DB available.

## Good Habits

- Enable memories once, then forget about them — the consolidation pipeline
  runs in the background between sessions, not mid-turn.
- Keep `no_memories_if_mcp_or_web_search = true` for threads that touch secrets
  or untrusted content.
- Treat memories as ambient background context, not a structured knowledge
  base — if you need durable project facts, put them in `AGENTS.md`.
- Reset memories if you switch roles or change working domains meaningfully —
  stale memories hurt more than they help.
- For CI / throwaway workflows, set `[features] memories = false` or use a
  separate `CODEX_HOME`.

## Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Using `[memory]` (singular) in config | The table is `[memories]` (plural) |
| Expecting memories to scope per project | Memories live under `$CODEX_HOME` — use `AGENTS.md` for project context |
| Storing secrets and expecting cleanup | Use `no_memories_if_mcp_or_web_search = true` and reset on exposure |
| Hand-editing `$CODEX_HOME/memories/*.md` | Reset via `/memories` — the pipeline owns these files |
| Editing memory extension resources manually | They're transient — cleanup runs them off every 7 days |
