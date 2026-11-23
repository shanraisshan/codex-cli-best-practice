# Best Practice: AGENTS.md

The `AGENTS.md` file is the primary instructions file for Codex CLI — providing project-level context and behavioral directives.

## File Naming and Fallbacks

Codex loads the first non-empty instruction file in each directory:
1. `AGENTS.override.md`
2. `AGENTS.md`
3. Names configured in `project_doc_fallback_filenames`, in order

Use `AGENTS.md` for checked-in guidance and `AGENTS.override.md` for an override at the same level. `CODEX.md` is only a fallback if you configure it.

## Sizing: Keep It Under 150 Lines

Keep instructions focused. This repo uses **150 lines** as an editorial guideline. Codex enforces the combined project budget in bytes, with a default of 32 KiB via `project_doc_max_bytes`.

**If you exceed 150 lines**, extract detailed content into:
- Skill files (`skills/<name>/SKILL.md`) for specialized procedures
- Separate docs files referenced by path

## Hierarchy and Override Mechanism

AGENTS.md files follow directory hierarchy:

```
/repo/AGENTS.md              # Root-level instructions
/repo/packages/api/AGENTS.md # Package-specific overrides
/repo/packages/web/AGENTS.md # Package-specific overrides
```

**Loading behavior**:
- Codex reads global guidance from `$CODEX_HOME`, then checks the path from the project root down to the working directory. Without a project root it checks only the working directory
- More specific (deeper) files take precedence over general (higher) ones
- All files are concatenated into context, with deeper files appearing later

## Recommended Structure

```markdown
# AGENTS.md

## Repository Overview
One paragraph describing what this project is and does.

## Key Components
Brief descriptions of major subsystems, with file paths.

## Critical Patterns
Non-obvious conventions the model MUST follow.

## Workflow Rules
Build, test, lint commands. Deployment patterns.

## Do NOT
Explicit anti-patterns to avoid.
```

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| Dumping entire API docs | Consumes the byte budget; dilutes focus | Link to docs; use skills |
| Repeating obvious conventions | Wastes lines on things the model knows | Only document the non-obvious |
| Long code examples | Eats line budget fast | Keep examples under 10 lines |
| Vague instructions ("be careful") | Not actionable | Be specific: "Always run `npm test` before committing" |
| Contradictory rules | Model picks one arbitrarily | Audit for conflicts |

## Monorepo Strategy

For monorepos, use a layered approach:

1. **Root `AGENTS.md`**: Shared conventions (git workflow, CI commands, coding standards)
2. **Package `AGENTS.md`**: Package-specific build commands, architecture decisions, testing patterns
3. **Skills**: Extract complex procedures (deployment, migration) into skills that any AGENTS.md can reference

This keeps each file short while maintaining comprehensive coverage.

## Truncation Behavior

When project instructions hit the configured byte budget:
- Later content can be truncated or omitted. Reserve space for instructions from deeper directories
- Put the most critical instructions at the top
- Use clear section headers so the model can scan structure even if details are lost
- Test by asking the model to repeat instructions from different sections

See [official AGENTS.md documentation](https://learn.chatgpt.com/docs/agent-configuration/agents-md#how-codex-discovers-guidance).
