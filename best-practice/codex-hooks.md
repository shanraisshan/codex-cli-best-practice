# Best Practice: Hooks

Hooks are an extensibility framework that inject custom scripts into the Codex agentic loop — enabling deterministic automation for logging, security scanning, validation, conversation summarization, and context-aware prompting.

<table width="100%">
<tr>
<td><a href="../">← Back to Codex CLI Best Practice</a></td>
<td align="right"><img src="../!/codex-jumping.svg" alt="Codex" width="60" /></td>
</tr>
</table>

> **Status:** Experimental — under active development. Windows support temporarily disabled.

## Feature Flag

Hooks require enabling in `config.toml`:

```toml
[features]
codex_hooks = true
```

## Discovery Locations

Codex discovers `hooks.json` files at two levels — both load simultaneously; higher-precedence layers don't replace lower-precedence hooks:

| Priority | Location | Scope |
|----------|----------|-------|
| 1 | `.codex/hooks.json` | Project (team-shared) |
| 2 | `~/.codex/hooks.json` | Global (personal) |

## Hook Events

| Event | Matcher | Description |
|-------|---------|-------------|
| `SessionStart` | `startup \| resume \| clear` | Runs at session initialization; `clear` distinguishes `/clear`-recreated sessions (v0.120.0+) |
| `PreToolUse` | `Bash` | Intercepts tool execution before running (Bash only) |
| `PostToolUse` | `Bash` | Reviews tool results after execution (Bash only) |
| `UserPromptSubmit` | Not supported | Runs when user submits a prompt |
| `Stop` | Not supported | Runs when a turn completes — determines whether to continue |

## Configuration Structure

Hooks organize into three levels: **event → matcher group → hook handlers**

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "pattern|regex",
        "hooks": [
          {
            "type": "command",
            "command": "script_path",
            "statusMessage": "optional UI feedback",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

### Key Options

| Option | Default | Description |
|--------|---------|-------------|
| `timeout` / `timeoutSec` | 600s | Execution time limit in seconds |
| `statusMessage` | — | Optional UI feedback during execution |
| `matcher` | Match all | Regex to filter event firing (`"*"`, `""`, or omit for all) |

## Runtime Behavior

- Matching hooks from multiple files all execute
- Multiple command hooks for the same event launch **concurrently**
- One hook cannot prevent another from running
- Commands run with session `cwd` as working directory

## Hook Events Deep Dive

### SessionStart

Injects context at session initialization. The `source` input field tells you
which lifecycle event triggered the hook:

| `source` | Meaning |
|---|---|
| `startup` | Fresh process start (`codex`) |
| `resume` | `/resume` or `codex resume <id>` |
| `clear` | Session recreated by `/clear` in the TUI (v0.120.0+) |

Branch on `source` if your context should differ — e.g. inject a "welcome to
this repo" preamble on `startup` but skip it on `clear` so `/clear` stays fast.

**Input fields:** `source`, `session_id`, `transcript_path`, `cwd`, `hook_event_name`, `model`

**Output:** Plain text on stdout is added as developer context. JSON output supports:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "text added as context"
  }
}
```

### PreToolUse

Intercepts tool execution before running (currently Bash only).

> **Note:** The model can circumvent this by writing and executing scripts directly — treat as a useful guardrail rather than a complete enforcement boundary.

**Input fields:** `turn_id`, `tool_name`, `tool_use_id`, `tool_input.command`, plus common fields

**Deny execution:**

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "reason text"
  }
}
```

**Alternative:** Exit code `2` with blocking reason on stderr.

### PostToolUse

Reviews tool results after execution (Bash only). Cannot undo side effects but can replace the tool result with feedback.

**Input fields:** `turn_id`, `tool_name`, `tool_use_id`, `tool_input.command`, `tool_response`, plus common fields

**Block and replace result:**

```json
{
  "decision": "block",
  "reason": "feedback reason",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "context text"
  }
}
```

**Alternative:** Exit code `2` with feedback reason on stderr.

### UserPromptSubmit

Runs when user submits a prompt. Matcher not supported.

**Input fields:** `turn_id`, `prompt`, plus common fields

**Block submission:**

```json
{
  "decision": "block",
  "reason": "reason text"
}
```

**Add context:**

```json
{
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "additionalContext": "context text"
  }
}
```

### Stop

Runs when a turn completes — determines whether to continue automatically. Matcher not supported.

**Input fields:** `turn_id`, `stop_hook_active`, `last_assistant_message`, plus common fields

**Continue with automatic prompt:**

```json
{
  "decision": "block",
  "reason": "continuation reason text"
}
```

> `decision: "block"` tells Codex to **continue** (not reject). The reason becomes the next prompt text. If any matching hook returns `continue: false`, that takes precedence.

## Common Input Fields

Every command hook receives JSON on stdin:

| Field | Type | Description |
|-------|------|-------------|
| `session_id` | string | Session/thread ID |
| `transcript_path` | string \| null | Path to session transcript |
| `cwd` | string | Working directory |
| `hook_event_name` | string | Current event name |
| `model` | string | Active model slug |
| `turn_id` | string | Turn-scoped hooks only |

## Common Output Fields

`SessionStart`, `UserPromptSubmit`, and `Stop` support:

| Field | Type | Description |
|-------|------|-------------|
| `continue` | boolean | `false` marks hook as stopped |
| `stopReason` | string | Recorded as stop reason |
| `systemMessage` | string | Surfaced as UI warning |
| `suppressOutput` | boolean | Parsed, not yet implemented |

Exit `0` with no output is treated as success — Codex continues normally.

## Path Resolution

For repo-local hooks, prefer git-root-based paths to avoid issues when Codex starts from subdirectories:

```
/usr/bin/python3 "$(git rev-parse --show-toplevel)/.codex/hooks/script.py"
```

## Full Example

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/.codex/hooks/session_start.py",
            "statusMessage": "Loading session notes"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use_policy.py\"",
            "statusMessage": "Checking Bash command"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/post_tool_use_review.py\"",
            "statusMessage": "Reviewing Bash output"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/user_prompt_submit.py\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python3 \"$(git rev-parse --show-toplevel)/.codex/hooks/stop_continue.py\"",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Relying on `PreToolUse` as a security boundary | Treat as a guardrail — the model can write scripts to bypass it |
| Using relative paths in hook commands | Use `$(git rev-parse --show-toplevel)` for stability |
| Missing the `[features]` flag | Always enable `codex_hooks = true` in `config.toml` |
| Setting very long timeouts on blocking hooks | Keep timeouts short to avoid stalling the agent loop |
| Assuming hooks can undo `PostToolUse` side effects | They can only replace the result, not reverse the action |
| Not handling JSON stdin properly | Every hook receives JSON on stdin — parse it correctly |
