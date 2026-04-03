# Best Practice: Skills

Skills are the primary mechanism for extending Codex CLI with reusable, composable instruction packages. They follow the `SKILL.md` standard and work best when they are narrow, trigger cleanly, and keep context small.

## Core Pattern

Codex uses skills in two ways:

1. Explicit invocation: mention the skill directly in the prompt. In the CLI or IDE, use `/skills` or type `$` to insert a skill mention.
2. Implicit invocation: Codex chooses the skill when the task matches the skill `description`.

That makes the `description` field the most important part of the skill metadata.

## Minimal Frontmatter

Use only the required frontmatter unless you have a documented reason to add more:

```yaml
---
name: docs-helper
description: Verify framework API details before making code changes or writing migration guidance.
---
```

The rest of the skill should focus on inputs, workflow, and expected outputs.

## Structure for Progressive Disclosure

Organize skills so Codex can load extra detail only when needed:

```text
.agents/skills/
  docs-helper/
    SKILL.md
    references/
    scripts/
    assets/
    agents/
      openai.yaml
```

- Put core workflow instructions in `SKILL.md`.
- Put deep reference material in `references/`.
- Put deterministic helpers in `scripts/`.
- Use `agents/openai.yaml` for optional UI metadata, invocation policy, and tool dependencies.

## Write Better Descriptions

Good descriptions act like trigger conditions:

```yaml
# Good
description: Review TypeScript changes for type-safety regressions and missing runtime validation.

# Bad
description: Help with TypeScript.
```

Be specific about when the skill should trigger and, when useful, when it should not.

## Design for Current Codex Behavior

- Treat skills as reusable workflows, not as custom slash commands.
- Prefer `/skills` and `$skill-name` mentions for explicit invocation examples.
- Use plugins to distribute reusable skills beyond a single repo.
- Use `[[skills.config]]` in `~/.codex/config.toml` to disable a skill without deleting it.

Example:

```toml
[[skills.config]]
path = "/path/to/skill/SKILL.md"
enabled = false
```

## Optional Metadata

Use `agents/openai.yaml` when you need richer metadata:

```yaml
interface:
  display_name: "Docs Helper"
  short_description: "Checks APIs before code changes"

policy:
  allow_implicit_invocation: false
```

This is the right place for UI presentation and invocation policy. Do not rely on stale or undocumented frontmatter fields.

## Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Treating skills as `/skill-name` slash commands | Document explicit invocation via `/skills` or `$skill-name` mentions |
| Using undocumented frontmatter like `user-invocable`, `allowed-tools`, or `context: fork` | Stick to current documented `SKILL.md` + `agents/openai.yaml` patterns |
| Treating `skills = [...]` as the main way to attach domain knowledge | Let skills trigger by description, or manage availability with `[[skills.config]]` |
| Putting all instructions in `AGENTS.md` | Extract focused workflows into skills |
| Writing broad descriptions that match everything | Make `description` a precise trigger condition |
