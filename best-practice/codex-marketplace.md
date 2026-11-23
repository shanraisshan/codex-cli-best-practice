# Best Practice: Plugin Marketplace

Codex CLI v0.121.0 introduced a first-class plugin marketplace — a way to
register third-party or internal plugin catalogs and install plugins from
them. A marketplace is a git repo (or local directory) containing a
`marketplace.json` manifest that lists plugins and how to fetch them.

<table width="100%">
<tr>
<td><a href="../">← Back to Codex CLI Best Practice</a></td>
<td align="right"><img src="../!/codex-jumping.svg" alt="Codex" width="60" /></td>
</tr>
</table>

## CLI Commands

All marketplace commands live under `codex plugin marketplace`:

```bash
codex plugin marketplace add <SOURCE> [--ref <REF>] [--sparse <PATH>]...
codex plugin marketplace list [--json]
codex plugin marketplace upgrade [MARKETPLACE_NAME]
codex plugin marketplace remove <MARKETPLACE_NAME>
```

`add` accepts several source forms:

| Source | Example |
|---|---|
| GitHub shorthand | `codex plugin marketplace add owner/repo` |
| Git URL | `codex plugin marketplace add https://github.com/owner/repo.git` |
| SSH URL | `codex plugin marketplace add git@github.com:owner/repo.git` |
| Local directory | `codex plugin marketplace add /abs/path/to/marketplace` |
| Relative path | `codex plugin marketplace add ./vendor/internal-market` |

Use `codex plugin marketplace list` to inspect the current catalogs and roots. Use `codex plugin list` to browse plugins from available catalogs, `codex plugin add` to install one, and `codex plugin remove` to uninstall it. Use `--help` for selection syntax.

### Partial clones

Use `--sparse` (repeatable) to fetch only the subdirectories you need:

```bash
codex plugin marketplace add openai/marketplace-big \
  --sparse plugins/toolkit \
  --sparse skills/release-notes
```

Codex runs `git clone --filter=blob:none --no-checkout` and pins the listed paths
via `git sparse-checkout`.

## `marketplace.json` Manifest

Codex looks for the manifest at one of:

- `.agents/plugins/marketplace.json` (Codex-native; API-key catalogs can use `.agents/plugins/api_marketplace.json`)
- `.claude-plugin/marketplace.json` (Claude Code compatible)
- `.cursor-plugin/marketplace.json` (Cursor compatible)

Minimal manifest:

```json
{
  "name": "my-marketplace",
  "plugins": [
    { "name": "release-notes", "source": "./plugins/release-notes" }
  ]
}
```

Manifest with several supported source forms:

```json
{
  "name": "codex-curated",
  "interface": { "displayName": "Codex Curated" },
  "plugins": [
    {
      "name": "local-plugin",
      "source": "./plugins/local-plugin"
    },
    {
      "name": "remote-subdir",
      "source": {
        "source": "git-subdir",
        "url": "openai/joey_marketplace",
        "path": "plugins/toolkit",
        "ref": "main"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL",
        "products": ["codex_cli"]
      },
      "category": "dev-tools"
    },
    {
      "name": "remote-url",
      "source": {
        "source": "url",
        "url": "https://github.com/owner/repo.git",
        "ref": "main"
      }
    },
    {
      "name": "npm-plugin",
      "source": {
        "source": "npm",
        "package": "@example/codex-toolkit",
        "version": "1.2.3"
      }
    }
  ]
}
```

### Field Reference

| Field | Required | Notes |
|---|---|---|
| `name` | yes | Marketplace key — also the directory name under `$CODEX_HOME` |
| `interface.displayName` | no | Label shown in `/plugins` |
| `plugins[].name` | yes | Plugin key |
| `plugins[].source` | yes | Bare string (local path) or object discriminated by `"source"` |
| `plugins[].policy.installation` | no | `NOT_AVAILABLE \| AVAILABLE \| INSTALLED_BY_DEFAULT` (default `AVAILABLE`) |
| `plugins[].policy.authentication` | no | `ON_INSTALL \| ON_USE` |
| `plugins[].policy.products` | no | Product metadata, e.g. `["codex_cli"]`. Use workspace policy for enforcement |
| `plugins[].category` | no | Taxonomy string for the UI |

`source` object types:

- `"local"` — `{ "source": "local", "path": "./..." }`
- `"url"` — `{ "source": "url", "url": "...", "path"?, "ref"?, "sha"? }`
- `"git-subdir"` — `{ "source": "git-subdir", "url": "...", "path": "...", "ref"?, "sha"? }` (path required)
- `"npm"` — `{ "source": "npm", "package": "...", "version"?, "registry"? }`

## On-Disk Layout

Codex caches marketplace contents and installed plugins under `$CODEX_HOME`
(usually `~/.codex/`):

```
$CODEX_HOME/
├── config.toml                   # [marketplaces.<name>] entries
├── .tmp/marketplaces/
│   └── <marketplace_name>/       # cloned manifest root
└── plugins/cache/
    └── <marketplace_name>/
        └── <plugin_name>/
            └── <version>/        # "local" when unversioned
                └── .codex-plugin/plugin.json
```

## `config.toml` Bookkeeping

`codex plugin marketplace add` records the marketplace in `config.toml` — you
rarely edit these by hand, but they look like this:

```toml
[marketplaces.my-marketplace]
source_type   = "git"
source        = "https://github.com/owner/repo.git"
ref           = "main"
sparse_paths  = ["plugins/toolkit"]
last_updated  = "2026-04-18T00:00:00Z"
last_revision = "abc123..."
```

`upgrade` re-runs `git ls-remote` against each entry and re-clones only if the
SHA has moved.

## Running Plugin Workflows

After installation, open `/plugins` to inspect a plugin's capabilities. Use the plugin skill mention shown in the composer to invoke a skill. Plugins can also bundle MCP servers and lifecycle hooks; inspect the manifest and review any hooks before trusting them.

## Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Using `codex marketplace add ...` (top-level) | The command is `codex plugin marketplace add` |
| Hand-editing `[marketplaces.*]` in config.toml | Use `add` / `upgrade` / `remove` — Codex manages the keys |
| Publishing marketplaces without `marketplace.json` at a supported path | Use `.agents/plugins/marketplace.json` for a new Codex catalog |
| Relying on `source: "./outside"` paths that escape the marketplace root | Local sources must stay under the marketplace directory |
| Cloning giant marketplace repos fully | Use `--sparse <PATH>` to fetch only what you need |

See the [plugin guide](https://learn.chatgpt.com/docs/plugins) and [tagged marketplace implementation](https://github.com/openai/codex/blob/rust-v0.171.0/codex-rs/core-plugins/src/marketplace.rs).
