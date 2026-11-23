# Best Practice: Subagents

Subagents let Codex spawn specialized agents in parallel to tackle complex, highly parallel tasks — such as codebase exploration or multi-step feature plans — then collect their results in one response.

With subagent workflows you can also define custom agents with different model configurations and instructions depending on the task.

## When to Use Subagents

Subagents are ideal for tasks that are:
- **Highly parallel** — multiple independent review or exploration threads
- **Domain-specialized** — different tasks need different models, tools, or instructions
- **Large-scope** — reviewing a full PR across security, quality, tests, etc.

Ask directly for delegation or describe when to delegate in applicable project or skill instructions. Each subagent does its own model and tool work, so subagent workflows consume more tokens than single-agent runs.

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

- Use `/subagents` in the CLI to switch between active agent threads and inspect ongoing work
- Ask Codex directly to steer a running subagent, stop it, or close completed threads

## Approvals and Sandbox Controls

Subagents inherit your current sandbox policy.

- In interactive CLI sessions, approval requests can surface from inactive threads. The overlay shows the source thread label — press `o` to open that thread before you approve/reject.
- In non-interactive flows, actions needing new approval fail and Codex surfaces the error to the parent workflow.
- A role can request a narrower sandbox. Parent session constraints still apply.

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

Each file defines one agent. Codex discovers these files without an additional registration entry in project config.

### Required Fields

Every custom agent must define:

| Field | Type | Purpose |
|-------|------|---------|
| `name` | string | Agent name Codex uses when spawning |
| `description` | string | When Codex should use this agent |
| `developer_instructions` | string | Core behavior instructions |

### Optional Fields

Model and reasoning effort generally inherit from the parent when omitted. Common optional configuration includes:

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
| `agents.max_concurrent_threads_per_session` | Backend default | Cap concurrent agent threads per session |
| `agents.max_depth` | 1 (V1) | Nesting depth for the V1 agent backend |

Use the current concurrency key. The older `max_threads` and `job_max_runtime_seconds` settings are legacy; the tagged release does not expose the former CSV batch tool.

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
max_concurrent_threads_per_session = 6
max_depth = 1
```

**`.codex/agents/pr-explorer.toml`:**
```toml
name = "pr_explorer"
description = "Read-only codebase explorer for gathering evidence before changes are proposed."
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

See [official subagent guidance](https://learn.chatgpt.com/docs/agent-configuration/subagents) for role discovery, model inheritance, and the built-in agents.
