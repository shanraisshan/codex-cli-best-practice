# Best Practice: Hooks

Hooks run a command or call an MCP tool at points in a Codex session. In v0.171.0 they are enabled by default. The feature key is `[features].hooks`; `codex_hooks` is a deprecated alias.

<table width="100%">
<tr>
<td><a href="../">← Back to Codex CLI Best Practice</a></td>
<td align="right"><img src="../!/codex-jumping.svg" alt="Codex" width="60" /></td>
</tr>
</table>

## Where Hooks Load

| Scope | Standalone file |
|---|---|
| System (macOS/Linux) | `/etc/codex/hooks.json` |
| System (Windows) | `%ProgramData%\OpenAI\Codex\hooks.json` |
| User | `$CODEX_HOME/hooks.json` |
| Trusted project | `.codex/hooks.json` |

You can also declare `[hooks]` in `config.toml`, and installed plugins can bundle hooks. Matching definitions from different sources can all run. Use one format per layer to avoid duplicates.

Open `/hooks` to inspect and trust new or changed non-managed hooks. Trust is recorded for the exact definition; changing a command or its settings can require another review.

## When Hooks Run

| Moment | Events in v0.171.0 |
|---|---|
| Session or subagent starts | `SessionStart`, `SubagentStart` |
| User submits a prompt | `UserPromptSubmit` |
| Tool call or approval | `PreToolUse`, `PermissionRequest`, `PostToolUse` |
| Model request attempt | `PreModelRequest`, `PostModelRequest` |
| Context compaction | `PreCompact`, `PostCompact` |
| Agent or turn is finishing | `SubagentStop`, `Stop` |
| Main thread interruption or end | `Interrupt`, `SessionEnd` |

`Stop` runs at turn completion. Use `SessionEnd` for main-thread shutdown and `Interrupt` for active-turn interruptions.

## Example: SessionStart

Save this in `.codex/hooks.json`. The checked-in [weather repo hook example](../.codex/hooks.json) uses this shape and also demonstrates four other events.

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "^(startup|resume|clear)$",
        "hooks": [
          {
            "type": "command",
            "command": "python3 .codex/hooks/scripts/hooks.py --hook SessionStart",
            "timeout": 10,
            "statusMessage": "Loading project context"
          }
        ]
      }
    ]
  }
}
```

An event contains matcher groups, and each group contains handlers. A `SessionStart` matcher checks the `source`; possible values also include `compact` and `fork`. Tool events match the canonical hook tool name, such as `Bash`, `apply_patch`, or an MCP tool name.

## Input and Output

- Commands receive event-specific JSON on stdin, including `hook_event_name`, `session_id`, and `cwd`. Read the payload once and use `cwd` to resolve workspace paths when appropriate.
- Successful observer hooks can exit `0` without output. `SessionStart` can add developer context using plain stdout or JSON with `hookSpecificOutput.additionalContext`.
- Blocking and continuation behavior depends on the event. `PreToolUse` can deny a call. A `Stop` hook can request another pass, so check `stop_hook_active` before requesting continuation.
- Multiple matching commands can start concurrently. Keep each hook independent, use short timeouts, and avoid printing secrets into logs or model context.

For example, a `SessionStart` handler can send a structured update:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Use the project test commands from AGENTS.md."
  }
}
```

Use the schema for the event you handle. The same payload is not valid for every event.

See the [official event and output reference](https://learn.chatgpt.com/docs/hooks) and the [v0.171.0 hook schemas](https://github.com/openai/codex/tree/rust-v0.171.0/codex-rs/hooks/schema/generated).
