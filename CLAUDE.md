# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A best practices reference repository for **Codex CLI** (v0.118.0+) and **Claude Code**, demonstrating the **Agent → Skill** orchestration pattern through a weather data system example. This is a documentation and configuration reference, not a traditional application codebase.

Maintained by Shayan Raees (@shanraisshan). Companion repos: [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice), [codex-cli-hooks](https://github.com/shanraisshan/codex-cli-hooks), [claude-code-hooks](https://github.com/shanraisshan/claude-code-hooks).

## Architecture

The repo demonstrates four interconnected systems:

1. **Config** (`.codex/config.toml`) — TOML-based layered config with 5 profiles (conservative, development, trusted, ci, review), MCP server registration, and agent definitions. Project-level overrides user-level (`~/.codex/config.toml`); CLI flags override both.

2. **Agents** (`.codex/agents/*.toml`) — Registered under `[agents.<name>]` in config.toml with dedicated role files. The weather-agent fetches temperature from Open-Meteo API and delegates rendering to a skill.

3. **Skills** (`.agents/skills/<name>/SKILL.md`) — Reusable instruction packages with YAML frontmatter (`name`, `description`). Discovered progressively from cwd up to `/etc/codex/skills`. Invoked via `/skills`, `$skill-name`, or auto-triggered by description match.

4. **Hooks** — Event-driven Python scripts. Claude Code has 27 hooks (`.claude/settings.json` → `.claude/hooks/scripts/hooks.py`); Codex CLI has 5 (`.codex/hooks.json`). Hooks play audio feedback per event with cross-platform support (macOS/Linux/Windows).

### Orchestration Flow

```
User Prompt → weather-agent (fetches temp from Open-Meteo)
                → /weather-svg-creator skill (renders SVG card)
                    → orchestration-workflow/weather.svg
                    → orchestration-workflow/output.md
```

Run it: `codex` then prompt "Fetch the current weather for Dubai in Celsius and create the SVG weather card output using the repo."

## Key Directories

- `best-practice/` — 6 comprehensive guides: config, agents-md, skills, subagents, hooks, mcp
- `orchestration-workflow/` — Weather system example with flow diagram and generated outputs
- `docs/SKILLS.md` — Skills system reference
- `examples/` — Profile configs and CI/CD integration examples
- `AGENTS.md` — Project guidance loaded hierarchically by Codex (cwd to git root, capped at 32 KiB)

## Configuration Quick Reference

**Profiles** (`codex --profile <name>`):
| Profile | Model | Sandbox | Approval |
|---------|-------|---------|----------|
| conservative | o4-mini | read-only | untrusted |
| development | o4-mini | workspace-write | on-request |
| trusted | o3 | workspace-write | never |
| ci | o4-mini | read-only | never |
| review | o3 | read-only | on-request |

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
