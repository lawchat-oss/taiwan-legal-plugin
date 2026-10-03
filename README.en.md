# taiwan-legal-plugin

**English** · [繁體中文](README.md)

> This plugin lets Claude query Taiwan's public legal databases — court judgments, statutes, constitutional interpretations, agency interpretations and court resolutions — directly inside Cowork or Claude Code.

`taiwan-legal-plugin` is a [Claude Code](https://claude.com/claude-code) plugin marketplace that brings Taiwanese legal open-data sources into Anthropic's [Claude for Legal](https://github.com/anthropics/claude-for-legal) ecosystem:

- **Judicial Yuan judgment portal** (judgment.judicial.gov.tw) — full-text search, judgment retrieval and appeal history
- **National Regulation Database** (law.moj.gov.tw) — 11,700+ statutes and regulations, official English translations, amendment and effective dates
- **Constitutional Court records** (cons.judicial.gov.tw) — 釋字, 憲判字, Justices' opinions, citation graph, case-file filings and the pending docket
- **Administrative interpretations (函釋)** — 45 official systems (ministries, the DGPA, the Consumer Protection Committee, the Examination Yuan, Taipei and New Taipei City) and the Executive Yuan Gazette, plus TIPO examination guidelines; discontinued letters are flagged where the site marks them
- **Court resolutions and precedents** (legal.judicial.gov.tw) — Supreme Court resolutions, legal Q&A conferences, discontinued precedents, 院字 / 院解字, Grand Chamber rulings, curated judgments
- **Appeal and quasi-judicial decisions** — Executive Yuan, ministry and local appeals; FTC, unfair labour practice, civil-service protection, FSC sanctions, Control Yuan, lawyer discipline
- **Legislative materials** — legislative reasons and process, Legislative Yuan bills and gazette, draft regulations open for comment
- **Research materials** — judicial and MOJ statistics, sentencing statistics, Judicial Yuan research reports, the NCL periodical index, open-access law journals
- **Other legal texts** — local regulations, treaties and agreements, exchange rules

The underlying MCP server is the open-source [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db), which exposes **26 tools**, wrapped in four research skills — judgment search, statute lookup (with legislative materials and other legal texts), interpretation lookup (with constitutional case files, appeal and quasi-judicial decisions), and research materials (literature, statistics, sentencing).

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

Restart Claude Code. The underlying [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db) is **fetched from PyPI and run automatically by `uvx` on first query** (pinned version, bumped with each plugin release); the first judgment search that needs a browser **auto-installs Chromium** (~150 MB, one time). No manual `pip install` or `playwright install` required.

For first-time use, run:

```
/taiwan-legal:cold-start-interview
```

to set defaults (court levels, date window, citation style).

## Skills (v0.7)

| Skill | Purpose |
|---|---|
| `/taiwan-legal:cold-start-interview` | One-time setup for research defaults (court, date window, citation style) |
| `/taiwan-legal:judgment-search` | Search judgments / retrieve a specific case's full text by 字號 or URL, checking its appeal history before citing |
| `/taiwan-legal:statute-lookup` | Look up regulations by name, article, keyword or in English; legislative reasons and records; amendment tracking; local regulations, treaties and exchange rules |
| `/taiwan-legal:interpretation-lookup` | 釋字 / 憲判字 (citation graph, case files, pending docket), 函釋 and examination guidelines (validity checked before citing), 決議 / 座談 / 判例 / curated judgments, appeal and quasi-judicial decisions |
| `/taiwan-legal:research-materials` | Legal scholarship (research reports, journal articles, research projects), judicial and MOJ statistics, sentencing statistics |

## Positioning

This is an access layer that brings Taiwan's public legal data into Claude Code via MCP. The data itself is maintained by the Judicial Yuan, the Ministry of Justice and the other issuing agencies under their open-data policies; this plugin **does not modify source content**. For performance and offline availability the underlying MCP server bundles a small local cache of public records (e.g., Constitutional Court reasonings); all cached items were fetched directly from the official portals (cons.judicial.gov.tw, judgment.judicial.gov.tw, law.moj.gov.tw) and each response carries the source URL. Source data is excluded from copyright under Article 9(1)(1) of the ROC Copyright Act (official documents / statutes); the structured packaging is released under CC0 1.0 (see `DATA_LICENSE` in [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db)).

## Design principles

- **Cite faithfully.** Every result carries the source URL plus 字號 / article number; the skills are instructed not to paraphrase legal holdings without quoting the operative text.
- **No legal advice.** Every skill closes with a "this is not legal advice" note and routes operational legal decisions back to a licensed attorney.
- **Data layer vs. access layer.** Data belongs to its publishers. What we build is the tooling and integration.

## License

Code in this repository is released under the MIT License. Source data is provided by the original publishers (Judicial Yuan, Ministry of Justice and the other issuing agencies) under their respective open-data policies.

---

Maintained by [lawchat-oss](https://github.com/lawchat-oss). Contributions welcome.
