# taiwan-legal

Access layer for Taiwan legal open data. See the top-level [README](../README.md) (繁體中文) / [README.en.md](../README.en.md) (English) for installation and usage.

## Bundled MCP server

This plugin bundles a stdio MCP server declaration in `.mcp.json` that launches the `mcp-taiwan-legal-db` package via `uvx` (auto-fetched from PyPI on first use; Chromium auto-installed on the first browser-backed query). The only prerequisite is [uv](https://docs.astral.sh/uv/).

Source: [github.com/lawchat-oss/mcp-taiwan-legal-db](https://github.com/lawchat-oss/mcp-taiwan-legal-db) (MIT, 80-test suite, CI on Python 3.10/3.11/3.12).

## Skills

- `skills/cold-start-interview/SKILL.md` — one-time defaults setup
- `skills/judgment-search/SKILL.md` — 裁判書 search and retrieval
- `skills/statute-lookup/SKILL.md` — 法規 lookup

## Practice profile

`cold-start-interview` writes the user's defaults to `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md`. The other skills load that file at the start of every run.
