# codex-cli-best-practice
practice makes codex perfect

![updated with Codex CLI](https://img.shields.io/badge/updated_with_Codex_CLI-v0.118.0%20(Apr%2003%2C%202026%2011%3A04%20PM%20PKT)-white?style=flat&labelColor=555) <a href="https://github.com/shanraisshan/codex-cli-best-practice/stargazers"><img src="https://img.shields.io/github/stars/shanraisshan/codex-cli-best-practice?style=flat&label=%E2%98%85&labelColor=555&color=white" alt="GitHub Stars"></a>

[![Best Practice](!/tags/best-practice.svg)](best-practice/) [![Implemented](!/tags/implemented.svg)](.codex/) [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) [![Codex](!/tags/codex.svg)](https://developers.openai.com/codex/overview) [![Community](!/tags/community.svg)](#-tips-and-tricks) ![Click on these badges below to see the actual sources](!/tags/click-badges.svg)<br>
<img src="!/tags/a.svg" height="14"> = Agents · <img src="!/tags/c.svg" height="14"> = Commands · <img src="!/tags/s.svg" height="14"> = Skills

<p align="center">
  <img src="!/codex-jumping.svg" alt="Codex CLI mascot jumping" width="120" height="100">
</p>

## 🧠 CONCEPTS

| Feature | Location | Description |
|---------|----------|-------------|
| <img src="!/tags/c.svg" height="14"> [**Commands**](https://developers.openai.com/codex/cli/slash-commands) | `custom not supported` | Custom commands (`.codex/commands/`) are not yet supported — 29 built-in slash commands: `/plan`, `/skills`, `/fast`, `/fork`, `/review`, `/apps`, and more |
| <img src="!/tags/a.svg" height="14"> [**Subagents**](https://developers.openai.com/codex/subagents) | [`.codex/agents/<name>.toml`](.codex/agents/) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-subagents.md) [![Implemented](!/tags/implemented.svg)](.codex/agents/) Custom agents registered under `[agents.<name>]` with dedicated TOML role configs, CSV batch processing, and multi-agent orchestration · Built-in: `default`, `worker`, `explorer` |
| <img src="!/tags/s.svg" height="14"> [**Skills**](https://developers.openai.com/codex/skills) | [`.agents/skills/<name>/SKILL.md`](.agents/skills/) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-skills.md) [![Implemented](!/tags/implemented.svg)](.agents/skills/) [Reference](docs/SKILLS.md) Reusable instruction packages with YAML frontmatter — invoke with `/skill-name` or preload into agents · Built-in: `$plan`, `$skill-creator`, `$web-search` · Distributed via [Plugins](https://developers.openai.com/codex/plugins) |
| [**Plugins**](https://developers.openai.com/codex/plugins) | `.codex-plugin/plugin.json` | Distributable bundles combining skills + app integrations + MCP servers — local/personal [marketplace](https://developers.openai.com/codex/plugins/build) system · Built-in: `$plugin-creator` · Browse via `/plugins` or Codex App |
| [**Workflows**](https://developers.openai.com/codex/workflows/) | [`.codex/agents/weather-agent.toml`](.codex/agents/weather-agent.toml) | [![Orchestration Workflow](!/tags/orchestration-workflow.svg)](orchestration-workflow/orchestration-workflow.md) End-to-end usage patterns — explain codebase, fix bugs, write tests, prototype from screenshot, iterate UI, delegate to cloud, code review, update docs |
| [**MCP Servers**](https://developers.openai.com/codex/mcp) | `config.toml` → `[mcp_servers.*]` | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-mcp.md) [![Implemented](!/tags/implemented.svg)](.codex/config.toml) Model Context Protocol for external tools — STDIO + Streamable HTTP servers · OAuth support (`codex mcp login`) |
| [**Config**](https://developers.openai.com/codex/config-basic) | [`.codex/config.toml`](.codex/config.toml) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-config.md) [![Implemented](!/tags/implemented.svg)](.codex/config.toml) TOML-based layered config system · [Profiles](https://developers.openai.com/codex/config-basic) · [Sandbox](https://developers.openai.com/codex/cli/features) · [Approval Policy](https://developers.openai.com/codex/cli/features) · [Advanced](https://developers.openai.com/codex/config-advanced) (`[features]`, `[otel]`, `[shell_environment_policy]`, `[tui]`, model providers, granular approvals) |
| [**Rules**](https://developers.openai.com/codex/rules) | `.codex/rules/` | Starlark-based command execution policies — `allow`, `prompt`, `forbidden` decisions with pattern matching · Test via `codex execpolicy check` · [AGENTS.override.md](https://developers.openai.com/codex/guides/agents-md) for personal instruction overrides |
| [**AGENTS.md**](https://developers.openai.com/codex/guides/agents-md) | [`AGENTS.md`](AGENTS.md) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-agents-md.md) Project-level context for Codex CLI — hierarchical discovery from cwd to repo root, capped at 32 KiB (`project_doc_max_bytes`) · `AGENTS.override.md` for personal overrides |
| [**Hooks**](https://developers.openai.com/codex/hooks) ![beta](!/tags/beta.svg) | [`.codex/hooks.json`](.codex/) | [![Best Practice](!/tags/best-practice.svg)](best-practice/codex-hooks.md) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/codex-cli-hooks) User-defined shell scripts that inject into the agentic loop — logging, security scanning, validation, and custom automation · Requires `codex_hooks = true` feature flag |
| [**Speed**](https://developers.openai.com/codex/speed) | `config.toml` → `service_tier` | Fast Mode (1.5x speed, 2x credits) on gpt-5.4 — toggle with `/fast on\|off\|status` · GPT-5.3-Codex-Spark for near-instant iteration (Pro subscribers) |
| [**Multi-Agent**](https://developers.openai.com/codex/multi-agent/) | `config.toml` → `[agents]` | Spawn specialized sub-agents in parallel — fan-out work, collect results, synthesize · `max_threads` (default 6), `max_depth` (default 1) · GA (`multi_agent = true` by default) |
| **AI Terms** | | [![Best Practice](!/tags/best-practice.svg)](https://github.com/shanraisshan/claude-code-codex-cursor-gemini/blob/main/reports/ai-terms.md) Agentic Engineering · Context Engineering · Vibe Coding |
| [**Best Practices**](https://developers.openai.com/codex/learn/best-practices) | | Official best practices · [Prompt Engineering](https://platform.openai.com/docs/guides/prompt-engineering) · [Codex Guides](https://developers.openai.com/codex/overview) |

<p align="center">
  <img src="!/codex-jumping.svg" alt="section divider" width="60" height="50">
</p>

[![Orchestration Workflow](!/tags/orchestration-workflow-hd.svg)](orchestration-workflow/orchestration-workflow.md)

See [orchestration-workflow](orchestration-workflow/orchestration-workflow.md) for implementation details of <img src="!/tags/a.svg" height="14"> **Agent** → <img src="!/tags/s.svg" height="14"> **Skill** pattern. The agent fetches temperature from Open-Meteo and invokes the SVG creator skill.

<p align="center">
  <img src="!/orchestration-workflow-diagram.svg" alt="Orchestration Workflow: Agent → Skill → Output" width="100%">
</p>

![How to Use](!/tags/how-to-use.svg)

```bash
codex
> Fetch the current weather for Dubai in Celsius and create the SVG weather card output using the repo.
```

> **Note:** This workflow is not 100% in sync with the [Claude Code Best Practice](https://github.com/shanraisshan/claude-code-best-practice) orchestration workflow. Codex CLI does not yet support custom commands (`.codex/commands/`), so the full <img src="!/tags/c.svg" height="14"> **Command** → <img src="!/tags/a.svg" height="14"> **Agent** → <img src="!/tags/s.svg" height="14"> **Skill** pattern is not possible. There is an experimental `tool/requestUserInput` in the Codex App Server docs and an internal `request_user_input` capability gated behind an under-development feature flag in codex-cli 0.115.0, but neither is publicly available yet.

<p align="center">
  <img src="!/codex-jumping.svg" alt="section divider" width="60" height="50">
</p>

## ⚙️ DEVELOPMENT WORKFLOWS

All major workflows converge on the same architectural pattern: **Research → Plan → Execute → Review → Ship**

| Name | ★ | Uniqueness | Plan | <img src="!/tags/a.svg" height="14"> | <img src="!/tags/s.svg" height="14"> |
|------|---|------------|------|---|---|
| [Superpowers](https://github.com/obra/superpowers) | 134k | ![TDD-first](https://img.shields.io/badge/TDD--first-ddf4ff) ![Iron Laws](https://img.shields.io/badge/Iron_Laws-ddf4ff) ![agent-agnostic](https://img.shields.io/badge/agent--agnostic-ddf4ff) | <img src="!/tags/s.svg" height="14"> [writing-plans](https://github.com/obra/superpowers/tree/main/skills/writing-plans) | 5 | 14 |
| [Spec Kit](https://github.com/github/spec-kit) | 85k | ![spec-driven](https://img.shields.io/badge/spec--driven-ddf4ff) ![constitution](https://img.shields.io/badge/constitution-ddf4ff) ![--ai-skills](https://img.shields.io/badge/----ai--skills-ddf4ff) | <img src="!/tags/s.svg" height="14"> [$speckit-plan](https://github.com/github/spec-kit) | 0 | 0 |
| [gstack](https://github.com/garrytan/gstack) | 63k | ![role personas](https://img.shields.io/badge/role_personas-ddf4ff) ![/codex review](https://img.shields.io/badge/%2Fcodex_review-ddf4ff) ![parallel sprints](https://img.shields.io/badge/parallel_sprints-ddf4ff) | <img src="!/tags/s.svg" height="14"> [autoplan](https://github.com/garrytan/gstack/tree/main/autoplan) | 0 | 31 |
| [Get Shit Done](https://github.com/gsd-build/get-shit-done) | 47k | ![fresh 200K contexts](https://img.shields.io/badge/fresh_200K_contexts-ddf4ff) ![wave execution](https://img.shields.io/badge/wave_execution-ddf4ff) ![--codex flag](https://img.shields.io/badge/----codex_flag-ddf4ff) | <img src="!/tags/a.svg" height="14"> [gsd-planner](https://github.com/gsd-build/get-shit-done/blob/main/agents/gsd-planner.md) | 21 | 0 |
| [oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex) | 23k | ![teams orchestration](https://img.shields.io/badge/teams_orchestration-ddf4ff) ![tmux workers](https://img.shields.io/badge/tmux_workers-ddf4ff) ![omc team N:codex](https://img.shields.io/badge/omc_team_N%3Acodex-ddf4ff) | <img src="!/tags/s.svg" height="14"> [ralplan](https://github.com/Yeachan-Heo/oh-my-codex/tree/main/skills/ralplan) | 19 | 36 |
| [Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) | 13k | ![Compound Learning](https://img.shields.io/badge/Compound_Learning-ddf4ff) ![Multi-Platform CLI](https://img.shields.io/badge/Multi--Platform_CLI-ddf4ff) ![install --to codex](https://img.shields.io/badge/install_----to_codex-ddf4ff) | <img src="!/tags/s.svg" height="14"> [ce-plan](https://github.com/EveryInc/compound-engineering-plugin/tree/main/plugins/compound-engineering/skills/ce-plan) | 49 | 42 |

### Others
- [Cross-Model (Claude Code + Codex) Workflow](https://github.com/shanraisshan/claude-code-best-practice/blob/main/development-workflows/cross-model-workflow/cross-model-workflow.md) [![Implemented](!/tags/implemented.svg)](https://github.com/shanraisshan/claude-code-best-practice/blob/main/development-workflows/cross-model-workflow/cross-model-workflow.md)

<p align="center">
  <img src="!/codex-jumping.svg" alt="section divider" width="60" height="50">
</p>

## 💡 TIPS AND TRICKS

![Community](!/tags/community.svg)

■ **Planning (2)**
- use [`/plan`](https://developers.openai.com/codex/cli/slash-commands) when you want an explicit plan — Codex may also plan automatically for multi-step tasks
- use [cross-model](https://github.com/shanraisshan/claude-code-best-practice/blob/main/development-workflows/cross-model-workflow/cross-model-workflow.md) (e.g., Claude Code) to review your plan before execution

■ **Workflows (8)**
- keep [`AGENTS.md`](https://developers.openai.com/codex/guides/agents-md) concise — 150 lines is a useful heuristic, but the actual limit is byte-based
- use [skills](https://developers.openai.com/codex/skills) with clear `name` and `description` frontmatter for auto-discovery
- use [`AGENTS.override.md`](https://developers.openai.com/codex/rules) for personal preferences without affecting the team
- use [profiles](https://developers.openai.com/codex/config-basic) to switch between project-defined safety levels — in this repo, `conservative` and `trusted` are examples
- use the built-in skill creator to scaffold new skills, and document one invocation style consistently across the repo
- start with [`on-request`](https://developers.openai.com/codex/cli/features) approval policy — only escalate to `never` when confident
- use [`--fork`](https://developers.openai.com/codex/cli/features) to explore alternatives without losing your session, [`--resume`](https://developers.openai.com/codex/cli/features) to pick up where you left off
- commit often — as soon as a task is completed, commit

■ **Workflows Advanced (4)**
- use [multi-agent](https://developers.openai.com/codex/multi-agent/) to spawn sub-agents for parallel fan-out work (GA — enabled by default)
- use [`codex exec`](https://developers.openai.com/codex/noninteractive) for headless/CI pipelines
- combine [sandbox modes](https://developers.openai.com/codex/cli/features) with [approval policies](https://developers.openai.com/codex/cli/features) — `workspace-write` + `on-request` is a good default
- [git worktrees](https://git-scm.com/docs/git-worktree) for parallel development

■ **Debugging (4)**
- always ask Codex to run the terminal (you want to see logs of) as a background task for better debugging
- use MCP ([Chrome DevTools](https://developer.chrome.com/blog/chrome-devtools-mcp), [Playwright](https://github.com/microsoft/playwright-mcp)) to let Codex see browser console logs on its own
- make it a habit to take screenshots and share with Codex whenever you are stuck with any issue
- use a different model for QA — e.g. [Claude Code](https://github.com/shanraisshan/claude-code-best-practice) for plan and implementation review

■ **Utilities (4)**
- [iTerm](https://iterm2.com/) terminal instead of IDE (crash issue)
- [Wispr Flow](https://wisprflow.ai) for voice prompting (10x productivity)
- [codex-cli-hooks](https://github.com/shanraisshan/codex-cli-hooks) for Codex feedback
- explore `config.toml` features like [profiles](https://developers.openai.com/codex/config-basic), [sandbox modes](https://developers.openai.com/codex/cli/features), and [MCP](https://developers.openai.com/codex/mcp) for a personalized experience

■ **Daily (2)**
- update Codex CLI daily and start your day by reading the [changelog](https://github.com/openai/codex/releases)
- follow [Tibo](https://x.com/thsottiaux), [Embiricos](https://x.com/embirico), [Jason](https://x.com/jxnlco), [Romain](https://x.com/romainhuet), [Dominik](https://x.com/dkundel), [Fouad](https://x.com/fouadmatin), [Bolin](https://x.com/bolinfest), [OpenAI Devs](https://x.com/OpenAIDevs) on X

![Codex](!/tags/codex.svg)

- Codex CLI — open-source local coding agent, first look (Fouad + Romain) | Apr 2025 ● [Tweet](https://x.com/OpenAIDevs/status/1912556874211422572)
- AMA with Codex team — CLI, sandbox, agents (Embiricos, Fouad, Tibo + team) | May 2025 ● [Reddit](https://www.reddit.com/r/ChatGPT/comments/1ko3tp1/ama_with_openai_codex_team/)
- Skills in Codex — standardizing .agents/skills across agents (Embiricos) | Feb 2026 ● [Tweet](https://x.com/embirico/status/2002102889653924111)
- Unrolling the Codex agent loop — how Codex works internally (Bolin) | Jan 2026 ● [Tweet](https://x.com/OpenAIDevs/status/2014794871962533970)
- How Codex is built — 90% self-built in Rust (Tibo, Pragmatic Engineer) | 17 Feb 2026 ● [Post](https://newsletter.pragmaticengineer.com/p/how-codex-is-built)
- Dogfood — Codex team uses Codex to build Codex (Tibo, Stack Overflow) | 24 Feb 2026 ● [Podcast](https://stackoverflow.blog/2026/02/24/dogfood-so-nutritious-it-s-building-the-future-of-sdlcs/)
- Why humans are AI's biggest bottleneck — Codex product vision (Embiricos, Lenny's) | Feb 2026 ● [Podcast](https://www.lennysnewsletter.com/p/why-humans-are-ais-biggest-bottleneck)
- How Codex team uses their coding agent (Tibo + Andrew, Every) | 18 Feb 2026 ● [Podcast](https://every.to/podcast/transcript-how-openai-s-codex-team-uses-their-coding-agent)

<p align="center">
  <img src="!/codex-jumping.svg" alt="section divider" width="60" height="50">
</p>

<a href="https://github.com/shanraisshan/claude-code-best-practice#billion-dollar-questions"><img src="!/tags/billion-dollar-questions.svg" alt="Billion-Dollar Questions"></a>

<p align="center">
  <img src="!/codex-jumping.svg" alt="section divider" width="60" height="50">
</p>

## Other Repos

<a href="https://github.com/shanraisshan/codex-cli-hooks"><img src="!/codex-speaking.svg" alt="Codex CLI Hooks" width="40" height="40" align="center"></a> <a href="https://github.com/shanraisshan/codex-cli-hooks"><strong>codex-cli-hooks</strong></a> · <a href="https://github.com/shanraisshan/claude-code-best-practice"><img src="!/claude-jumping.svg" alt="Claude Code" width="40" height="40" align="center"></a> <a href="https://github.com/shanraisshan/claude-code-best-practice"><strong>claude-code-best-practice</strong></a> · <a href="https://github.com/shanraisshan/claude-code-hooks"><img src="!/claude-speaking.svg" alt="Claude Code Hooks" width="40" height="40" align="center"></a> <a href="https://github.com/shanraisshan/claude-code-hooks"><strong>claude-code-hooks</strong></a>

---

<a href="https://openai.com/form/codex-for-oss/"><img src="!/tags/codex-for-oss.svg" alt="Codex for Open Source" width="720"></a>
