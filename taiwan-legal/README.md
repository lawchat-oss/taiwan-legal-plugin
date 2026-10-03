# taiwan-legal

Access layer for Taiwan legal open data. See the top-level [README](../README.md) (繁體中文) / [README.en.md](../README.en.md) (English) for installation and usage.

## Bundled MCP server

This plugin bundles a stdio MCP server declaration in `.mcp.json` that launches the `mcp-taiwan-legal-db` package (with its `[captcha]` extra) via `uvx`, pinned to an exact server version that is bumped with each plugin release (fetched from PyPI on first use; Chromium is installed automatically on the first query that needs a browser). The only prerequisite is [uv](https://docs.astral.sh/uv/).

Source: [github.com/lawchat-oss/mcp-taiwan-legal-db](https://github.com/lawchat-oss/mcp-taiwan-legal-db) (MIT, CI on Python 3.10/3.11/3.12).

## Skills

- `skills/cold-start-interview/SKILL.md` — one-time defaults setup
- `skills/judgment-search/SKILL.md` — 裁判書 search and retrieval
- `skills/statute-lookup/SKILL.md` — 法規 lookup, including 立法理由
- `skills/interpretation-lookup/SKILL.md` — 釋字 / 憲判字 and case files, 行政函釋, 決議 / 法律問題座談 / 判例, 訴願 and quasi-judicial decisions
- `skills/research-materials/SKILL.md` — legal scholarship, official statistics and sentencing statistics

## Practice profile

`cold-start-interview` writes the user's defaults to `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md`. The other skills load that file at the start of every run.
