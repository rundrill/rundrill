# RunDrill plugin marketplace

The public **install index** for RunDrill's programming & professional courses. Each course ships
as one plugin — a `<subject>-coach` skill backed by an MCP drill engine (curriculum, progress,
mistake memory).

Natural-language courses (English, Kazakh, …) live in the sibling
[`rundrill/rundrill-lang`](https://github.com/rundrill/rundrill-lang) marketplace.

This repo holds only the catalogs; the course plugins live in their own repos under the
[`rundrill`](https://github.com/rundrill) org and are referenced by Git source. Two catalogs cover
the two marketplace-based hosts (one entry per course in each):

- `.claude-plugin/marketplace.json` — **Claude Code / Desktop**, `github` source.
- `.agents/plugins/marketplace.json` — **OpenAI Codex**, `url` source.

Authoring scaffolding (`_template/`) lives in the private development monorepo, not here.

## Installing (for users)

**Claude Code / Claude Desktop** — via this marketplace:
```
/plugin marketplace add rundrill/rundrill        # the org/repo of this index
/plugin install rundrill-python@rundrill             # <plugin-name>@<marketplace-name>
/reload-plugins                                      # → /python-coach is live
```
`@rundrill` is the marketplace `name` field (set in `marketplace.json`), not the repo name.
(Language courses install from the other index: `rundrill-english@rundrill-lang`.)

**OpenAI Codex** — via this marketplace too:
```
codex plugin marketplace add rundrill/rundrill    # GitHub shorthand; reads .agents/plugins/marketplace.json
```
Then open the plugin directory, pick the **RunDrill** marketplace, and install the course.

**Google Antigravity** — no marketplace; Antigravity scans plugin dirs. Drop a course folder in:
- `~/.gemini/config/plugins/rundrill-python/` (global, all workspaces), or
- `<workspace>/.agents/plugins/rundrill-python/` (workspace-scoped).

## Registering a published course

A course plugin lives in its own repo (e.g. `rundrill/rundrill-python`). Add one entry to **each**
catalog, pinned to a released tag (`ref`, and optionally a full-commit `sha`) so users don't pull
unfinished work.

`.claude-plugin/marketplace.json` (Claude — `github` source):
```json
{
  "name": "rundrill-python",
  "source": { "source": "github", "repo": "rundrill/rundrill-python", "ref": "v0.2.11" },
  "description": "Python coach — read and review code (incl. AI-generated) with drills, progress, and mistake memory."
}
```

`.agents/plugins/marketplace.json` (Codex — `url` source; always include `policy` + `category`):
```json
{
  "name": "rundrill-python",
  "source": { "source": "url", "url": "https://github.com/rundrill/rundrill-python.git", "ref": "v0.2.11" },
  "policy": { "installation": "AVAILABLE", "authentication": "ON_INSTALL" },
  "category": "Education"
}
```

## Naming convention

| Layer | Pattern | Example | Notes |
|---|---|---|---|
| Plugin (install unit) | `rundrill-<subject>` | `rundrill-english` | unique within the marketplace; the Antigravity plugin dir name too |
| Skill / slash command | `<subject>-coach` | `/english-coach` | subject-distinct so the bare command works; Claude falls back to `/rundrill-english:english-coach` on a clash |
| MCP server | `rundrill-<subject>` | server key in both `.mcp.json` and `mcp_config.json` | per-course name; one shared backend behind per-course endpoints |

Rules: lowercase **kebab-case** everywhere (not underscores); hyphenate multi-word subjects
(`system-design`, `soft-skills`); lowercase acronyms (`toefl`, `ielts`, `owasp`). The SKILL.md
`name:` field, its folder name, and the slash command must all match.

Planned regional bundles use a distinct word so they never collide with a single course:
`rundrill-pack-west`, `rundrill-pack-asia` (thin meta-plugins listing course plugins as deps).

## Notes

- A plugin folder must be **self-contained** — its skills and MCP config can't reference files
  outside the plugin, because users install one plugin at a time. Shared harness logic lives
  server-side in the MCP backend, not in the plugin.
- **Remote MCP shapes differ:** Claude uses `{ "type": "http", "url": … }` in `.mcp.json`;
  Antigravity uses `{ "serverUrl": … }` in `mcp_config.json`. If RunDrill's OAuth server supports
  **dynamic client registration (DCR)**, Antigravity needs only `serverUrl` — it handles the OAuth
  flow itself. Otherwise add `oauth: { clientId, clientSecret }` and register
  `https://antigravity.google/oauth-callback` as a redirect URI.

Authoring a new course and the submodule layout: see the private monorepo's `plugins/README.md`
and [Plugin Packaging & Distribution](https://github.com/kmmbvnr/rundrill/blob/main/wiki/plugin-packaging.md).
