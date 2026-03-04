# codex-cli-best-practice
practice makes codex perfect

![Last Updated](https://img.shields.io/badge/Last_Updated-Mar_04%2C_2026_02%3A17_PM_PKT-white?style=flat&labelColor=555) <a href="https://github.com/shanraisshan/codex-cli-best-practice/stargazers"><img src="https://img.shields.io/github/stars/shanraisshan/codex-cli-best-practice?style=flat&label=%E2%98%85&labelColor=555&color=white" alt="GitHub Stars"></a>

[![Best Practice](!/tags/best-practice.svg)](best-practice/) *Click on this badge to show the latest best practice*<br>
[![Implemented](!/tags/implemented.svg)](.codex/) *Click on this badge to show implementation in this repo*<br>
[![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) *Click on this badge to see the Skill → Skill orchestration workflow*

<p align="center">
  <img src="!/codex-jumping.svg" alt="Codex CLI mascot jumping" width="120" height="100">
</p>

## CONCEPTS

| Feature | Location | Description |
|---------|----------|-------------|
| [**Instructions**](https://developers.openai.com/codex/guides/agents-md) | [`AGENTS.md`](AGENTS.md) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-agents-md.md) Project-level context for Codex CLI — loaded from every directory up to repo root, capped at 32 KiB. Fallback: `CODEX.md` |
| [**Configuration**](https://developers.openai.com/codex/config-basic) | [`.codex/config.toml`](.codex/config.toml) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-config.md) [![Implemented](!/tags/implemented.svg)](.codex/config.toml) TOML-based layered config: global (`~/.codex/`) → project (`.codex/`) → CLI flags |
| [**Profiles**](https://developers.openai.com/codex/config-basic) | [`.codex/config.toml`](.codex/config.toml) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-config.md) [![Implemented](!/tags/implemented.svg)](.codex/config.toml) Named config presets — `codex --profile ci`, `--profile conservative`, `--profile trusted` |
| [**Skills**](https://developers.openai.com/codex/skills) | [`.agents/skills/<name>/SKILL.md`](.agents/skills/) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-skills.md) [![Implemented](!/tags/implemented.svg)](.agents/skills/) [Reference](docs/SKILLS.md) Reusable instruction packages with YAML frontmatter — invoke with `/skill-name` or preload into agents |
| [**Built-in Skills**](https://developers.openai.com/codex/skills) | `$plan`, `$skill-creator`, `$web-search` | Skills shipped with Codex CLI — planning, skill generation, and web search |
| [**Sandbox**](https://developers.openai.com/codex/cli/features) | `config.toml` → `sandbox_mode` | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-sandbox.md) OS-level isolation: `read-only` (no writes), `workspace-write` (no network), `danger-full-access` |
| [**Approval Policy**](https://developers.openai.com/codex/cli/features) | `config.toml` → `approval_policy` | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-approval-policies.md) Autonomy levels: `untrusted` (ask everything), `on-request` (model decides), `never` (full auto) |
| [**MCP Servers**](https://developers.openai.com/codex/mcp) | `config.toml` → `[mcp_servers.*]` | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-mcp.md) [![Implemented](!/tags/implemented.svg)](.codex/config.toml) Model Context Protocol for external tools — plus Codex-as-MCP-server pattern |
| [**Agents**](https://developers.openai.com/codex/cli/features) | [`.codex/config.toml`](.codex/config.toml) + [`.codex/agents/<name>.toml`](.codex/agents/) | [![Implemented](!/tags/implemented.svg)](.codex/agents/) Specialized subagents registered under `[agents.<name>]`, optionally backed by dedicated TOML role configs |
| [**Headless / CI**](https://developers.openai.com/codex/noninteractive) | `codex exec "prompt"` | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-headless.md) Non-interactive execution for pipelines and scripts |
| [**Sessions**](https://developers.openai.com/codex/cli/features) | `--resume`, `--fork` | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-sessions.md) Persistent sessions: resume where you left off or fork to explore alternatives |
| [**Override**](https://developers.openai.com/codex/rules) | [`AGENTS.override.md`](AGENTS.override.md) | Personal instruction overrides — loaded before `AGENTS.md`, not committed to git |

[![Orchestration Workflow](!/tags/orchestration-workflow-hd.svg)](orchestration-workflow/orchestration-workflow.md)

See [orchestration-workflow](orchestration-workflow/orchestration-workflow.md) for full implementation details of the **Skill → Skill** sequential pipeline pattern.

<p align="center">
  <img src="!/orchestration-workflow-diagram.svg" alt="Orchestration Workflow: Skill → Skill → Output" width="100%">
</p>

| Component | Role | Location |
|-----------|------|----------|
| **weather-fetcher** | Fetches current temperature from Open-Meteo API | [`.agents/skills/weather-fetcher/SKILL.md`](.agents/skills/weather-fetcher/SKILL.md) |
| **weather-svg-creator** | Creates SVG weather card | [`.agents/skills/weather-svg-creator/SKILL.md`](.agents/skills/weather-svg-creator/SKILL.md) |
| **weather-agent** | Agent with preloaded weather-fetcher skill | [`.codex/agents/weather-agent.toml`](.codex/agents/weather-agent.toml) |
| **output.md** | Generated weather report | [`orchestration-workflow/output.md`](orchestration-workflow/output.md) |

## TIPS AND TRICKS

![Shayan](!/tags/shayan.svg)

■ **Workflows**
- Keep [`AGENTS.md`](https://developers.openai.com/codex/guides/agents-md) under 150 lines — content beyond this is increasingly ignored
- Use [skills](https://developers.openai.com/codex/skills) with clear `name` and `description` frontmatter for auto-discovery
- Avoid context degradation, do manual compacting at max 50%
- Use [profiles](https://developers.openai.com/codex/config-basic) to switch between safety levels (`conservative` for review, `trusted` for dev)
- Use `AGENTS.override.md` for personal preferences without affecting the team
- [git worktrees](https://git-scm.com/docs/git-worktree) for parallel development

■ **Daily**
- Update Codex CLI daily and start your day by reading the [changelog](https://github.com/openai/codex/blob/main/CHANGELOG.md)

■ **Hourly**
- Commit often — as soon as a task is completed, commit

■ **Utilities**
- [iTerm](https://iterm2.com/) terminal instead of IDE (crash issue)
- [Wispr Flow](https://wisprflow.ai) for voice prompting (10x productivity)
- [codex-cli-voice-hooks](https://github.com/shanraisshan/codex-cli-voice-hooks) for Codex feedback
- Explore `config.toml` features like [profiles](https://developers.openai.com/codex/config-basic), [sandbox modes](https://developers.openai.com/codex/cli/features), and [MCP](https://developers.openai.com/codex/mcp) for a personalized experience

■ **Debugging**
- Always ask Codex to run the terminal (you want to see logs of) as a background task for better debugging
- Use MCP ([Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp), [Playwright](https://github.com/microsoft/playwright-mcp)) to let Codex see browser console logs on its own
- Provide screenshots of the issue
- Use a different model for QA — e.g. [Claude Code](https://claude.com/claude-code) for plan and implementation review

## REPORTS

| Report | Description |
|--------|-------------|
| [Configuration Hierarchy](reports/config-hierarchy.md) | Config merge order, layer precedence, profile interaction, and environment variable expansion |
| [Sandbox Modes](reports/sandbox-modes.md) | OS-level isolation architecture, security analysis, and performance impact by mode |
| [Approval Policies](reports/approval-policies.md) | Policy mechanics, allow-list design patterns, and the sandbox + approval safety matrix |
| [MCP Patterns](reports/mcp-patterns.md) | Client/server MCP, agent-scoped servers, Codex-as-MCP-server, and sandbox interaction |
| [Headless CI/CD](reports/headless-ci-cd.md) | `codex exec` integration with GitHub Actions, GitLab CI, and Jenkins |

---

<table width="100%"><tr><td align="left"><a href="https://github.com/shanraisshan/claude-code-best-practice">← Back to Claude Code Best Practice</a></td><td align="right"><img src="!/claude-jumping.svg" alt="Claude jumping" width="60" height="54"></td></tr></table>
