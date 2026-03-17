# AGENTS.md

This file provides guidance to Codex CLI when working with code in this repository.

## Repository Overview

This is a best practices repository for OpenAI Codex CLI configuration, demonstrating patterns for skills, agents, orchestration workflows, and project-scoped configuration. It serves as a reference implementation rather than an application codebase.

## Key Components

### Weather System (Example Workflow)

A demonstration of the **Agent → Skill** orchestration pattern:
- `weather-agent` (`.codex/agents/weather-agent.toml`): Entry point — fetches temperature from Open-Meteo API, invokes renderer skill
- `weather-svg-creator` skill (`.agents/skills/weather-svg-creator/SKILL.md`): Invoked by agent — creates SVG weather card

The orchestration flow: agent fetches temperature from Open-Meteo (using caller-provided unit, defaults to Celsius), then invokes `/weather-svg-creator` to render the SVG output. See `orchestration-workflow/orchestration-workflow.md` for the complete flow diagram.

### Skill Definition Structure

Skills live in `.agents/skills/<name>/SKILL.md` and use YAML frontmatter:
- `name`: Display name (defaults to directory name)
- `description`: When to invoke the skill (used for auto-discovery)

Each skill directory may also contain:
- `scripts/`: Executable code the skill invokes
- `references/`: Documentation the skill references
- `assets/`: Templates, resources, or static files
- `agents/openai.yaml`: Optional appearance and dependency metadata

Codex discovers skills via progressive disclosure — it starts with metadata and loads full instructions only when a skill is activated.

### Configuration System

Codex CLI uses TOML-based configuration at two levels:
- **User-level**: `~/.codex/config.toml` — personal defaults across all projects
- **Project-level**: `.codex/config.toml` — team-shared, project-scoped overrides (loaded only when the project is trusted)

### Configuration Hierarchy

1. `.codex/config.toml`: Team-shared project settings (checked in)
2. `~/.codex/config.toml`: Personal user-level settings
3. CLI flags (`--model`, `--ask-for-approval`, `--sandbox`): Override both config files
4. `--config key=value`: One-off overrides from the command line

### Agents and Skills

Skills are discovered from multiple scopes in order of precedence:
1. `$CWD/.agents/skills` — current working directory (most specific)
2. `$CWD/../.agents/skills` — parent directories up to repo root
3. `$REPO_ROOT/.agents/skills` — repository root
4. `$HOME/.agents/skills` — user-level personal skills
5. `/etc/codex/skills` — system/admin-level shared skills

Agents are registered under `[agents.<name>]` in `.codex/config.toml` and can
optionally point to dedicated role files in `.codex/agents/*.toml`.

## AGENTS.md Discovery

Codex walks from the Git root down to the current working directory, loading `AGENTS.override.md` then `AGENTS.md` in each directory. Files closer to the current directory appear later in the combined prompt and take precedence. The combined size is capped at 32 KiB by default (`project_doc_max_bytes`).

## Profiles

Define named profiles in `config.toml` under `[profiles.<name>]` to switch between configurations quickly:

```bash
codex --profile conservative   # read-only, asks before every action
codex --profile development    # workspace-write sandbox, on-request approval
codex --profile trusted        # no approval prompts, workspace-write sandbox
codex --profile ci             # headless CI/CD mode
codex --profile review         # read-only code review mode
```

Set a default profile with `profile = "conservative"` at the top level of `config.toml`. Example profile configs are in `examples/profiles/`.

## Workflow Best Practices

From experience with this repository:

- Keep AGENTS.md under 150 lines for reliable adherence
- Use skills with clear `name` and `description` frontmatter for auto-discovery
- Organize skills by feature domain (e.g., `weather-svg-creator`)
- Use profiles to switch between safety levels (`conservative` for review, `trusted` for development)
- Use `AGENTS.override.md` for personal preferences without affecting the team
- Break complex tasks into composable skills rather than monolithic instructions

### Sandbox Modes

- `read-only`: Only reads files, no writes or network access
- `workspace-write`: Reads and writes within the project, sandboxed network
- `danger-full-access`: Unrestricted access (use with caution)

### Approval Policies

- `untrusted`: Only safe read commands auto-approved; everything else asks
- `on-request`: Model decides when to ask for approval (recommended default)
- `never`: All commands auto-approved; failures returned to model directly

## MCP Servers

MCP servers are configured under `[mcp_servers.*]` in `.codex/config.toml`. Currently configured:
- `context7`: Documentation lookup via `@upstash/context7-mcp@latest`

## Documentation

- `best-practice/codex-agents-md.md`: AGENTS.md authoring guide
- `best-practice/codex-config.md`: Config, profiles, and MCP layout
- `best-practice/codex-mcp.md`: MCP servers best practices
- `best-practice/codex-skills.md`: Skills best practices
- `best-practice/codex-subagents.md`: Subagents guide
- `docs/SKILLS.md`: Skills system reference
- `orchestration-workflow/orchestration-workflow.md`: Weather system flow diagram
- `examples/`: Example profile configs and CI/CD setup

