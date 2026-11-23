# Changelog

Tracks repository verification against stable Codex CLI releases. Use the [verification checklist](verification-checklist.md) for future refreshes.

## 2026-11-24 PKT · Codex CLI v0.171.0

**Baseline:** v0.137.0. **Target:** [v0.171.0](https://github.com/openai/codex/releases/tag/rust-v0.171.0), confirmed against the [latest stable release pointer](https://github.com/openai/codex/releases/latest) and the installed `codex-cli 0.171.0`. The companion Claude repo's Concepts checklist and changelog informed the audit format.

Reviewed the user-facing release notes from v0.138.0 through v0.171.0. Cross-checked the [official docs index](https://learn.chatgpt.com/llms.txt), [tagged config schema](https://github.com/openai/codex/blob/rust-v0.171.0/codex-rs/core/config.schema.json), [tagged slash commands](https://github.com/openai/codex/blob/rust-v0.171.0/codex-rs/tui/src/slash_command.rs), [tagged model catalog](https://github.com/openai/codex/blob/rust-v0.171.0/codex-rs/models-manager/models.json), and the installed CLI help and feature states.

| Release notes | Changes relevant to this repo |
|---|---|
| [v0.140.0](https://github.com/openai/codex/releases/tag/rust-v0.140.0) and [v0.145.0](https://github.com/openai/codex/releases/tag/rust-v0.145.0) | Usage view, session deletion, unified mentions, Claude and Cursor imports, and an opt-in second subagent backend |
| [v0.147.0](https://github.com/openai/codex/releases/tag/rust-v0.147.0) through [v0.150.0](https://github.com/openai/codex/releases/tag/rust-v0.150.0) | Automatic approval review flag, conversation export, command center, session queue, and asynchronous, MCP, and interrupt hooks |
| [v0.154.0](https://github.com/openai/codex/releases/tag/rust-v0.154.0) through [v0.157.0](https://github.com/openai/codex/releases/tag/rust-v0.157.0) | Managed worktrees, fullscreen terminal, and background server |
| [v0.163.0](https://github.com/openai/codex/releases/tag/rust-v0.163.0) through [v0.166.0](https://github.com/openai/codex/releases/tag/rust-v0.166.0) | Terminal preferences, saved presets, inline model and permission selections, and the v2 memory store on fresh memory-enabled profiles |
| [v0.169.0](https://github.com/openai/codex/releases/tag/rust-v0.169.0) and [v0.171.0](https://github.com/openai/codex/releases/tag/rust-v0.171.0) | Older-task search, default terminal workspace, GPT-6.2 catalog, remote control, and remote exec |

| README section | Disposition | Verification |
|---|---|---|
| Header and navigation | Updated | Release badge, current CLI guide, audit links, and section anchors |
| Concepts and Hot | Updated | Added Goals, permission profiles, worktrees, models, non-interactive mode, terminal workspace, remote controls, and import. Reconciled MCP, hooks, profiles, subagents, and memories with the tagged release |
| Orchestration Workflow | Updated | Matched the named agent and skill to tracked files, separated fetching from rendering, and kept the diagram and output paths current |
| Development Workflows | Updated | Replaced static GitHub star counts with live badges. Community workflow steps and capability counts remain editorial snapshots |
| Tips and Tricks | Updated | Refreshed Codex URLs and tips that referenced outdated hook flags, profile behavior, and memory settings. Retained attributed community advice |
| Videos and Podcasts | Historical | Dated references retained |
| Subscribe | Historical | Curated channels and profiles retained |
| How to Use | Added | Links to setup, the working examples, and the audit history |
| Other Repos | Verified | Reviewed companion links and the local Claude reference checkout |
| Sponsor My Work | Historical | Existing sponsorship links and badges retained |

### Guide and Example Notes

- Migrated legacy `[profiles.*]` examples to named user config files and aligned both project instruction files. Preserved sandbox boundaries while removing retired approval policies and old pinned models.
- Updated the subagent guide to use `/subagents`, standalone roles, and the current concurrency key. Removed the former CSV tool example after checking the tagged source. Updated skill discovery and metadata.
- Documented the [removal of the Codex MCP server](https://learn.chatgpt.com/docs/mcp-server) and retained outbound MCP configuration. Updated plugin CLI commands, hook trust and supported events, and local memory migration.
- Updated the CI example to pin the audited CLI and use the [official Codex Action](https://learn.chatgpt.com/docs/github-action) with read-only sandboxing and a separate config. The sample analyzes test failures for projects with a Node lockfile.

### Validation

- Checked 144 local link occurrences and 60 added or changed external links, including anchor fragments. Reviewed the final diff and ran `git diff --check`.
- Parsed the TOML, hook JSON, workflow YAML, and SVG assets. Checked the action inputs against [the v1 action definition](https://github.com/openai/codex-action/blob/v1/action.yml) and checked shell syntax for all workflow run steps.
- Ran strict config startup and loaded the three example profiles in a temporary checkout with the v0.171.0 CLI. Discovered the weather skill and five project hooks through the local app server. Checked the hook handler with startup and clear fixtures.
- Verified the Open-Meteo Celsius and Fahrenheit response fields. The authenticated model run that regenerates the weather card and the hosted GitHub Actions jobs were not run. Audio playback was not part of the hook fixture.
