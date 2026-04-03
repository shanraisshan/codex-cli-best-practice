# Skills System Reference

Skills are reusable instruction packages that extend Codex CLI with focused workflows and domain expertise. They follow the open `SKILL.md` standard and are the authoring format behind reusable Codex workflows.

## Skill Structure

Skills live in `.agents/skills/<name>/` and must include `SKILL.md`. A skill directory can also include supporting material for progressive disclosure:

```text
.agents/skills/
  my-skill/
    SKILL.md
    scripts/
    references/
    assets/
    agents/
      openai.yaml
```

- `SKILL.md`: Required instructions plus metadata
- `scripts/`: Optional executable helpers
- `references/`: Optional docs and examples
- `assets/`: Optional templates or static resources
- `agents/openai.yaml`: Optional UI, policy, and dependency metadata

## Minimal `SKILL.md`

Codex requires only `name` and `description` in YAML frontmatter:

```markdown
---
name: my-skill
description: Explain exactly when this skill should and should not trigger.
---

# My Skill

Instructions Codex should follow when this skill is activated.
```

The `description` is the trigger surface for implicit invocation, so write it as a precise "when should this fire?" statement.

## How Codex Uses Skills

Codex can activate a skill in two ways:

1. Explicit invocation: mention the skill directly in your prompt. In the CLI or IDE, use `/skills` or type `$` to insert a skill mention.
2. Implicit invocation: Codex chooses the skill when your task matches the skill `description`.

Codex uses progressive disclosure for skills. It starts with metadata such as `name`, `description`, path, and optional `agents/openai.yaml` data, then loads the full `SKILL.md` only when the skill is selected.

## Built-in Skills

Codex bundles system skills that are available out of the box. Common examples include:

- `$plan`
- `$skill-creator`
- `$skill-installer`

Built-in skill inventory can evolve across releases, so prefer examples over hard-coded lists.

## Discovery Paths

Codex discovers skills from these locations:

1. Repository skills from the current working directory up to the repository root: `.agents/skills/`
2. User skills: `~/.agents/skills/`
3. Admin skills: `/etc/codex/skills`
4. System skills bundled with Codex

Codex scans repository locations from the current working directory upward. If two skills share the same name, Codex does not merge them.

## Distribute Skills with Plugins

Skills are the authoring format. Plugins are the installable distribution unit for reusable skills, apps, and MCP integrations.

Use direct skill folders for repo-local workflows and day-to-day authoring. Package skills as plugins when you want to distribute them, bundle them with apps or MCP config, or publish them through a marketplace.

## Enable or Disable Skills

Use `[[skills.config]]` entries in `~/.codex/config.toml` to disable a skill without deleting it:

```toml
[[skills.config]]
path = "/path/to/skill/SKILL.md"
enabled = false
```

Restart Codex after changing skill config.

## Optional Metadata with `agents/openai.yaml`

Use `agents/openai.yaml` for optional UI, policy, and dependency metadata instead of relying on undocumented frontmatter fields:

```yaml
interface:
  display_name: "Docs Helper"
  short_description: "Verifies framework APIs before code changes"

policy:
  allow_implicit_invocation: false

dependencies:
  tools:
    - type: mcp
      value: openaiDeveloperDocs
      description: OpenAI Docs MCP server
      transport: streamable_http
      url: https://developers.openai.com/mcp
```

## Best Practices

- Keep each skill focused on one job.
- Prefer instructions over scripts unless you need deterministic behavior or external tooling.
- Write `description` fields as trigger conditions, not marketing copy.
- Use `references/` and `scripts/` to keep `SKILL.md` concise.
- Use plugins for reusable distribution outside a single repo.
