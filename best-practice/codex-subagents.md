# Best Practice: Subagents

Subagents let Codex spawn specialized agents in parallel to tackle complex, highly parallel tasks — such as codebase exploration or multi-step feature plans — then collect their results in one response.

With subagent workflows you can also define custom agents with different model configurations and instructions depending on the task.

## When to Use Subagents

Subagents are ideal for tasks that are:
- **Highly parallel** — multiple independent review or exploration threads
- **Domain-specialized** — different tasks need different models, tools, or instructions
- **Large-scope** — reviewing a full PR across security, quality, tests, etc.

Codex only spawns subagents when you explicitly ask it to. Each subagent does its own model and tool work, so subagent workflows consume more tokens than single-agent runs.

## Typical Workflow

Codex handles orchestration: spawning subagents, routing follow-up instructions, waiting for results, and closing threads. When many agents are running, Codex waits until all results are available before returning a consolidated response.

**Try this prompt on your project:**

```
I would like to review the following points on the current PR (this branch vs main).
Spawn one agent per point, wait for all of them, and summarize the result for each point.
1. Security issue
2. Code quality
3. Bugs
4. Race conditions
5. Test flakiness
6. Maintainability of the code
```

## Managing Subagents

- Use `/agent` in the CLI to switch between active agent threads and inspect ongoing work
- Ask Codex directly to steer a running subagent, stop it, or close completed threads

## Approvals and Sandbox Controls

Subagents inherit your current sandbox policy.

- In interactive CLI sessions, approval requests can surface from inactive threads. The overlay shows the source thread label — press `o` to open that thread before you approve/reject.
- In non-interactive flows, actions needing new approval fail and Codex surfaces the error to the parent workflow.
- Codex reapplies the parent turn's live runtime overrides (sandbox, `/approvals` changes, `--yolo`) when spawning children, even if a custom agent file sets different defaults.
- You can override sandbox config per custom agent (e.g., mark one as read-only).

## Built-in Agents

Codex ships with three built-in agents:

| Agent | Purpose |
|-------|---------|
| `default` | General-purpose fallback |
| `worker` | Execution-focused for implementation and fixes |
| `explorer` | Read-heavy codebase exploration |

## Custom Agents

Define custom agents as standalone TOML files:
- `~/.codex/agents/` — personal agents
- `.codex/agents/` — project-scoped agents

Each file defines one agent and can override the same settings as a normal Codex session config.

### Required Fields

Every custom agent must define:

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Agent name Codex uses when spawning |
| `description` | string | When Codex should use this agent |
| `developer_instructions` | string | Core behavior instructions |

### Optional Fields

These inherit from the parent session when omitted:

- `nickname_candidates` — display nicknames for spawned instances
- `model` — model override
- `model_reasoning_effort` — reasoning effort level
- `sandbox_mode` — sandbox override (e.g., `read-only`)
- `mcp_servers` — MCP server connections
- `skills.config` — skill configurations

### Global Subagent Settings

Configure in your `config.toml` under `[agents]`:

| Field | Default | Purpose |
|-------|---------|---------|
| `agents.max_threads` | 6 | Concurrent open agent thread cap |
| `agents.max_depth` | 1 | Spawned agent nesting depth (root = 0) |
| `agents.job_max_runtime_seconds` | 1800 | Default timeout per worker for CSV jobs |

### Display Nicknames

Use `nickname_candidates` for readable labels when running many instances of the same agent:

```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
"""
nickname_candidates = ["Atlas", "Delta", "Echo"]
```

Nicknames are presentation-only — Codex still identifies agents by `name`.

### Best Practices for Custom Agents

The best custom agents are **narrow and opinionated**:
- Give each one a clear, single job
- Match the tool surface to that job
- Write instructions that prevent drifting into adjacent work
- If a custom agent name matches a built-in (e.g., `explorer`), the custom agent takes precedence

## Example: PR Review Pattern

Split review across three focused agents:

**Project config (`.codex/config.toml`):**
```toml
[agents]
max_threads = 6
max_depth = 1
```

**`.codex/agents/pr-explorer.toml`:**
```toml
name = "pr_explorer"
description = "Read-only codebase explorer for gathering evidence before changes are proposed."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Stay in exploration mode.
Trace the real execution path, cite files and symbols, and avoid proposing fixes
unless the parent agent asks for them.
Prefer fast search and targeted file reads over broad scans.
"""
```

**`.codex/agents/reviewer.toml`:**
```toml
name = "reviewer"
description = "PR reviewer focused on correctness, security, and missing tests."
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "read-only"
developer_instructions = """
Review code like an owner.
Prioritize correctness, security, behavior regressions, and missing test coverage.
Lead with concrete findings, include reproduction steps when possible,
and avoid style-only comments unless they hide a real bug.
"""
```

**`.codex/agents/docs-researcher.toml`:**
```toml
name = "docs_researcher"
description = "Documentation specialist that uses the docs MCP server to verify APIs and framework behavior."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Use the docs MCP server to confirm APIs, options, and version-specific behavior.
Return concise answers with links or exact references when available.
Do not make code changes.
"""

[mcp_servers.openaiDeveloperDocs]
url = "https://developers.openai.com/mcp"
```

**Prompt:**
```
Review this branch against main. Have pr_explorer map the affected code paths,
reviewer find real risks, and docs_researcher verify the framework APIs
that the patch relies on.
```

## Example: Frontend Integration Debugging

Three agents for UI regressions and cross-stack bugs:

**`.codex/agents/code-mapper.toml`:**
```toml
name = "code_mapper"
description = "Read-only codebase explorer for locating relevant frontend and backend code paths."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
sandbox_mode = "read-only"
developer_instructions = """
Map the code that owns the failing UI flow.
Identify entry points, state transitions, and likely files before the worker starts editing.
"""
```

**`.codex/agents/browser-debugger.toml`:**
```toml
name = "browser_debugger"
description = "UI debugger that uses browser tooling to reproduce issues and capture evidence."
model = "gpt-5.4"
model_reasoning_effort = "high"
sandbox_mode = "workspace-write"
developer_instructions = """
Reproduce the issue in the browser, capture exact steps, and report what the UI actually does.
Use browser tooling for screenshots, console output, and network evidence.
Do not edit application code.
"""

[mcp_servers.chrome_devtools]
url = "http://localhost:3000/mcp"
startup_timeout_sec = 20
```

**`.codex/agents/ui-fixer.toml`:**
```toml
name = "ui_fixer"
description = "Implementation-focused agent for small, targeted fixes after the issue is understood."
model = "gpt-5.3-codex-spark"
model_reasoning_effort = "medium"
developer_instructions = """
Own the fix once the issue is reproduced.
Make the smallest defensible change, keep unrelated files untouched,
and validate only the behavior you changed.
"""

[[skills.config]]
path = "/Users/me/.agents/skills/docs-editor/SKILL.md"
enabled = false
```

**Prompt:**
```
Investigate why the settings modal fails to save. Have browser_debugger reproduce it,
code_mapper trace the responsible code path, and ui_fixer implement the smallest fix
once the failure mode is clear.
```

## Path-Based Agent Addressing (v0.121.0+)

Every agent has a canonical path from the root thread. Spawned children extend
the parent path:

```
/root                       # the root session
/root/task1                 # spawned from root
/root/task1/reviewer        # spawned from /root/task1
```

Segments are lowercase letters, digits, and underscores only. `root`, `.`, and
`..` are reserved.

Inside a given agent, siblings may be addressed by bare name (`reviewer`) OR
canonical path (`/root/task1/reviewer`); cross-branch targets must use the
canonical path.

This addressing is primarily tool-facing (not a command you type). Codex uses
it inside the structured messaging APIs below.

## Structured Inter-Agent Messaging (v0.121.0+)

The v2 agent tool set replaces the older v1 spawn/result shape:

| Tool | Purpose |
|---|---|
| `spawn_agent` | Spawn a child; returns `{ task_name, nickname }` |
| `send_message` | Enqueue a message to an existing agent (no turn triggered) |
| `followup_task` | Enqueue a message AND trigger a turn (`interrupt: true` preempts) |
| `wait_agent` | Wait for any mailbox update in the current tree (`timeout_ms` optional) |
| `list_agents` | List live agents, optionally filtered by `path_prefix` |
| `close_agent` | Close an agent and its descendants |
| `resume_agent` | Resume a previously closed agent by id |

Key behaviors to know:

- `spawn_agent` takes `fork_turns: "none" | "all" | <int>` to control context inheritance.
- `followup_task` cannot target `/root` — you cannot reassign the root thread.
- `wait_agent` returns `{ message, timed_out }` without message content — the
  content is fetched separately from the target's mailbox.
- `list_agents` returns `agent_status` values `pending_init | running | interrupted | shutdown | not_found | { completed } | { errored }`.

User-visible equivalents:

- `/agent` — switch between active agent threads in the TUI.
- `/resume` — jump to a session by ID or name (v0.119.0+).

## CSV Batch Processing (Experimental)

Use `spawn_agents_on_csv` for many similar tasks that map to one row per work item. Codex reads the CSV, spawns one worker per row, waits for the batch to finish, and exports combined results.

**Good for:**
- Reviewing one file, package, or service per row
- Checking lists of incidents, PRs, or migration targets
- Generating structured summaries for many similar inputs

**Parameters:**

| Parameter | Purpose |
|-----------|---------|
| `csv_path` | Source CSV |
| `instruction` | Worker prompt template with `{column_name}` placeholders |
| `id_column` | Column for stable item IDs |
| `output_schema` | JSON object shape each worker must return |
| `output_csv_path` | Export path |
| `max_concurrency` | Parallel worker limit |
| `max_runtime_seconds` | Per-worker timeout |

Each worker must call `report_agent_job_result` exactly once — if a worker exits without reporting, Codex marks that row with an error.

**Example prompt:**
```
Create /tmp/components.csv with columns path,owner and one row per frontend component.

Then call spawn_agents_on_csv with:
- csv_path: /tmp/components.csv
- id_column: path
- instruction: "Review {path} owned by {owner}. Return JSON with keys path, risk,
  summary, and follow_up via report_agent_job_result."
- output_csv_path: /tmp/components-review.csv
- output_schema: an object with required string fields path, risk, summary, and follow_up
```
