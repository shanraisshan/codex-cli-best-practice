# Verification Checklist

Use this checklist when refreshing the repository for a stable Codex CLI release.

| Check | Evidence to collect |
|---|---|
| Release target | Open the [latest stable GitHub release](https://github.com/openai/codex/releases/latest), pin the release tag, and record the date and installed `codex --version`. Skip prerelease tags. |
| Release changes | Read the user-facing notes since the last audited version. Track added, renamed, deprecated, and removed commands, features, and settings. Follow linked source or tests for ambiguous behavior. |
| Concepts and availability | Compare the README feature rows with the [official docs index](https://learn.chatgpt.com/llms.txt). Check features against the target tag's source, config schema, and built-in command list when the live docs and the CLI differ. Label gated, preview, experimental, and desktop-only features. |
| Guide consistency | Cross-check the README, `AGENTS.md`, `CLAUDE.md`, `best-practice/`, and `docs/` for repeated claims, feature flags, command names, defaults, and config precedence. |
| Links and badges | Follow changed external URLs and anchors. Verify local files, badges, anchor targets, and diagrams. Confirm each Implemented badge points to an actual example. |
| Configuration and examples | Parse TOML, JSON, YAML, and relevant assets. Compare fields with the pinned schema or action definition. Confirm the docs match tracked files. Check named profile filenames and precedence, hook event and handler shapes, and agent and skill discovery. |
| Runtime checks | Run safe CLI smoke checks with a temporary `CODEX_HOME` and checkout. Record what actually ran. For demos with external APIs, verify inputs and responses separately if a full run needs credentials, permission, or CI infrastructure. |
| Final review | Run `git diff --check`, review the diff for contradictions and unrelated changes, and account for every major README section. Update the version badge after verification and append an entry to the [changelog](changelog.md). |

Record each item as **Updated**, **Verified**, **Historical**, or **Unverified**. For unverified items, record the affected file or section, what was attempted, and what evidence is still needed.
