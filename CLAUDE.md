# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A best practices reference repository for **Codex CLI** (verified on v0.171.0) and **Claude Code**, demonstrating the **Agent → Skill** orchestration pattern through a weather data system example. This is a documentation and configuration reference, not a traditional application codebase.

Maintained by Shayan Raees (@shanraisshan). Companion repos: [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice), [codex-cli-hooks](https://github.com/shanraisshan/codex-cli-hooks), [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks).

## Architecture

The repo demonstrates four interconnected systems:

1. **Config** (`.codex/config.toml`) — Trusted project settings with an MCP integration. Named config profiles are separate user TOML files.

2. **Agents** (`.codex/agents/*.toml`) — Standalone roles. The weather agent fetches Dubai temperature from Open-Meteo and returns it to the parent.

3. **Skills** (`.agents/skills/<name>/SKILL.md`) — Codex discovers reusable instructions and exposes them with `/skills` and `$skill-name` mentions.

4. **Hooks** — Separate examples for Claude Code (`.claude/settings.json`) and Codex CLI (`.codex/hooks.json`). The Codex example registers five events.

### Orchestration Flow

```
User Prompt → weather-agent (fetches temp from Open-Meteo)
                → parent uses $weather-svg-creator (renders SVG card)
                    → orchestration-workflow/weather.svg
                    → orchestration-workflow/output.md
```

Run it: `codex` then prompt "Use weather-agent to fetch the current temperature in Dubai in Celsius, then use $weather-svg-creator to create the weather card."

## Key Directories

- `best-practice/` — 8 comprehensive guides: config, agents-md, skills, subagents, hooks, mcp, marketplace, memory
- `orchestration-workflow/` — Weather system example with flow diagram and generated outputs
- `docs/SKILLS.md` — Skills system reference
- `examples/` — Profile configs and CI/CD integration examples
- `AGENTS.md` — Project guidance loaded from project root to working directory with a default 32 KiB project budget

## Configuration Quick Reference

See [Config](best-practice/codex-config.md) for named profiles, precedence, and the CI example.

## Git Commit Convention

**One file, one commit** — do NOT bundle multiple file changes into a single commit. Each file gets its own commit with a descriptive message specific to that file's changes.

For example, if `README.md`, `best-practice/codex-agents-md.md`, and a skill file all changed:
- Commit 1: `git add README.md` → commit with README-specific message
- Commit 2: `git add best-practice/codex-agents-md.md` → commit with agents-doc-specific message
- Commit 3: `git add .agents/skills/weather-svg-creator/SKILL.md` → commit with skill-specific message

This keeps git history clean and makes it easy to review, revert, or cherry-pick individual changes.

## Content Guidelines

When editing best-practice guides or documentation in this repo:
- Keep AGENTS.md under 150 lines (32 KiB byte cap)
- Skill descriptions should be triggers ("when should I fire?"), not summaries
- Use profiles for safety switching; keep behavioral rules out of AGENTS.md when config.toml is deterministic
- Skills should stay under 150 lines with progressive disclosure (core in SKILL.md, details in `references/`)
