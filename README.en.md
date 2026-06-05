# taiwan-legal-plugin

**English** · [繁體中文](README.md)

> This plugin lets Claude query Taiwan's public legal databases — court judgments and statutes — directly inside Cowork or Claude Code.

`taiwan-legal-plugin` is a [Claude Code](https://claude.com/claude-code) plugin marketplace that brings three Taiwanese legal open-data sources into Anthropic's [Claude for Legal](https://github.com/anthropics/claude-for-legal) ecosystem:

- **Judicial Yuan judgment portal** (judgment.judicial.gov.tw) — full-text search and judgment retrieval
- **National Regulation Database** (law.moj.gov.tw) — 11,700+ statutes and regulations
- **Constitutional Court records** (cons.judicial.gov.tw, planned for v0.2)

The underlying MCP server is the open-source [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db), which exposes **eight tools**. v0.1 ships two skills wrapping a subset of those tools — judgment search and statute lookup. **Constitutional Court interpretations (釋字 / 憲判字) and citation-graph tools exist in the MCP server but are not yet exposed as skills — planned for v0.2.**

## Install

**Prerequisite:** install [uv](https://docs.astral.sh/uv/) (a single cross-platform binary; skip if you already have it):

```
curl -LsSf https://astral.sh/uv/install.sh | sh           # macOS / Linux
powershell -c "irm https://astral.sh/uv/install.ps1|iex"  # Windows
```

In Claude Code:

```
/plugin marketplace add github:lawchat-oss/taiwan-legal-plugin
/plugin install taiwan-legal@taiwan-legal-plugin
```

Restart Claude Code. The underlying [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db) is **fetched from PyPI and run automatically by `uvx` on first query**; the first judgment search that needs a browser **auto-installs Chromium** (~150 MB, one time). No manual `pip install` or `playwright install` required.

For first-time use, run:

```
/taiwan-legal:cold-start-interview
```

to set defaults (court levels, date window, citation style).

## Skills (v0.1)

| Skill | Purpose |
|---|---|
| `/taiwan-legal:cold-start-interview` | One-time setup for research defaults (court, date window, citation style) |
| `/taiwan-legal:judgment-search` | Search judgments / retrieve a specific case's full text by 字號 or URL |
| `/taiwan-legal:statute-lookup` | Look up regulations by name, article, or keyword |

Planned for v0.2: `taiwan-interpretation-lookup` (大法官解釋 / 憲判字 with citation graph).

## Positioning

This is an access layer that brings Taiwan's public legal data into Claude Code via MCP. The data itself is maintained by the Judicial Yuan and the Ministry of Justice under their open-data policies; this plugin **does not modify source content**. For performance and offline availability the underlying MCP server bundles a small local cache of public records (e.g., Constitutional Court reasonings); all cached items were fetched directly from the official portals (cons.judicial.gov.tw, judgment.judicial.gov.tw, law.moj.gov.tw) and each response carries the source URL. Source data is excluded from copyright under Article 9(1)(1) of the ROC Copyright Act (official documents / statutes); the structured packaging is released under CC0 1.0 (see `DATA_LICENSE` in [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db)).

## Design principles

- **Cite faithfully.** Every result carries the source URL plus 字號 / article number; the skills are instructed not to paraphrase legal holdings without quoting the operative text.
- **No legal advice.** Every skill closes with a "this is not legal advice" note and routes operational legal decisions back to a licensed attorney.
- **Data layer vs. access layer.** Data belongs to its publishers. What we build is the tooling and integration.

## License

Code in this repository is released under the MIT License. Source data is provided by the original publishers (Judicial Yuan, Ministry of Justice) under their respective open-data policies.

---

Maintained by [lawchat-oss](https://github.com/lawchat-oss). Contributions welcome.
