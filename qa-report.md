# Skills QA — self-evaluation report

> Author self-evaluation against the Legal Skill Design Framework v0.1 (13 design parameters, 3 legal-specific failure modes, 3-band verdict) as published in [`anthropics/claude-for-legal`](https://github.com/anthropics/claude-for-legal) `legal-builder-hub/skills/skills-qa/SKILL.md`. **End users should run `/legal-builder-hub:skills-qa` against the installed skill directory for an independent assessment** — this document is the author's pre-publication check, not a substitute for runtime QA.

Evaluated: **2026-10-04** (plugin v0.7.0, bundled server `mcp-taiwan-legal-db` 1.8)
Source: first-party (LawChat OSS)
Skills evaluated: `cold-start-interview`, `judgment-search`, `statute-lookup`, `interpretation-lookup`, `research-materials` (in `taiwan-legal/skills/`)

---

## Prompt-injection heuristic scan

Author self-scan against the 10 categories in the QA spec (override/ignore instructions, authority claims, config-override, out-of-scope reads, out-of-scope writes, external URLs, hidden content, shell/code execution, credential-adjacent asks, legal authority overclaiming).

**Findings**: none detected across the five SKILL.md files. The only file write is `cold-start-interview` writing `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` (in scope for plugin config); no hooks, no agents, no `Bash` / `WebFetch` / `WebSearch` tool grants, no encoded blobs, no zero-width characters or HTML directives. The only URLs in the skills are the maintainer's GitHub pages and one law.moj.gov.tw citation example. All data access goes through the declared `taiwan-legal-db` MCP server.

> This is a heuristic scan by the author, not a security audit. End users should rely on `/legal-builder-hub:skill-installer`'s independent scan and human approval gate.

---

## Dependency map

| Direction | What | Notes |
|---|---|---|
| Upstream | `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` | Practice profile written by `cold-start-interview`; read by the four lookup skills. Skills handle absence by offering to run cold-start or to proceed without defaults. |
| Upstream | `taiwan-legal-db` MCP server (bundled via `.mcp.json`) | stdio transport; launched with `uvx --from mcp-taiwan-legal-db[captcha]==<pinned version> mcp-taiwan-legal-db` (fetched from PyPI on first use). Requires `uv` on PATH. Chromium is installed automatically on the first query that needs a browser. |
| Upstream (via the server) | Official Taiwan legal sources | The server scrapes official sites live, on the user's machine, one user-initiated query at a time — more than 120 government and public-institution domains, listed in the server's `SOURCES.md`. Some sites disallow crawlers in robots.txt; for some the server completes JavaScript / Cloudflare checks in a local browser or reads image captchas with local OCR. No login, no privileged access, no bulk collection. See the server's Disclaimer. |
| Downstream | `cold-start-interview` writes profile; no other writes | No skill writes outside `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/`. |
| Auto-triggers | none | No hooks; no agents; no scheduled invocations. |
| Breakage risk | MCP server unavailable → every lookup skill surfaces the error and stops. A single upstream source failing is reported per source (`categories[].error`) while the others still return. | No silent fallback; "zero results" is never read as "nothing exists". |

---

## Parameter evaluation

### `cold-start-interview`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Stated: same audience as the lookup skills; config-only. |
| 2 | Work Shape             | ✅     | N/A — interactive configuration, not legal work; noted explicitly. |
| 3 | Delegation Threshold   | ✅     | N/A — no legal interpretation occurs. |
| 4 | Input Requirements     | ✅     | Defaults provided for every question; user may skip any. |
| 5 | Versioning / Ownership | ✅     | Plugin version in `plugin.json`; maintainer line in skill. |
| 6 | Confidence Bands       | ✅     | N/A — interactive configuration; declared. |
| 7 | Failure Modes          | ✅     | File-write failure handled; merge ambiguity handled; legal failure modes N/A and declared. |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — config-only, single-file write, no remote persistence. |
| 9 | Escalation Logic       | ✅     | N/A — no high-stakes legal work. |
| 10 | Trust Surface         | ✅     | Writes only the profile file; no hooks; no `Bash` / `WebFetch`. |
| 11 | Freshness             | ✅     | N/A — no bundled `references/` content. |
| 12 | Schema                | ✅     | Frontmatter complete (description 447 chars); six interview questions; worked example; scope/limitations; idempotency section. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills (US-focused). |

**Legal failure mode check**: N/A for all three (configuration only) — declared.

**Verdict**: **READY**

---

### `judgment-search`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Legal researchers, attorneys, paralegals, in-house counsel, law students working in/with the Taiwanese legal system. |
| 2 | Work Shape             | ✅     | Pattern-Matched Review — research/lookup with explicit non-summarization rule. |
| 3 | Delegation Threshold   | ✅     | Conclusions and recommendations are the user's (or their attorney's) responsibility — structural via the verbatim-quoting rule. |
| 4 | Input Requirements     | ✅     | Ambiguity handling (search vs. read full text → ask); profile fallback prompts cold-start. |
| 5 | Versioning / Ownership | ✅     | Plugin version + maintainer line. |
| 6 | Confidence Bands       | ✅     | High / Medium / Low explicit, with "do not synthesize" rule on Low. |
| 7 | Failure Modes          | ✅     | All three legal-specific modes addressed; appeal history (`history`) checked before citing so a reversed judgment is not cited as good law. |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — no advice, no summary without quote, no jurisdiction comparison, no document generation, refuse privileged content. |
| 9 | Escalation Logic       | ✅     | Specific deflect sentence. |
| 10 | Trust Surface         | ✅     | No hooks; MCP declared with source attribution; no off-skill file writes; no overclaiming. |
| 11 | Freshness             | ✅     | No bundled references; data fetched live from judgment.judicial.gov.tw at query time. |
| 12 | Schema                | ✅     | Frontmatter complete (description 598 chars); eight-step workflow; worked example with concrete tool calls and output shape; scope/limitations; confidence bands; failure modes; tools-used list. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills. |

**Legal failure mode check**:
- Legal advice vs. legal support: ✅ structural — verbatim quoting + "not legal advice" notice.
- Privilege implications: ✅ explicit — refuses inputs that look like privileged communications.
- Accountability gap: ✅ explicit — output is a research artifact, not a concluded answer.

**Verdict**: **READY**

---

### `statute-lookup`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Same audience as `judgment-search`; familiarity with Taiwanese statute structure (法 / 條 / 項 / 款 / 目). |
| 2 | Work Shape             | ✅     | Pattern-Matched Review — canonical lookup. |
| 3 | Delegation Threshold   | ✅     | Interpretation, application to facts and conclusions are the user's (or their attorney's) responsibility. |
| 4 | Input Requirements     | ✅     | Ambiguous names → disambiguation; articles requested by number, range or list (the server returns an outline, not the whole law, when none is given); profile fallback. |
| 5 | Versioning / Ownership | ✅     | Plugin version + maintainer line. |
| 6 | Confidence Bands       | ✅     | High / Medium / Low explicit; Low includes "do not invent or guess article content". |
| 7 | Failure Modes          | ✅     | All three legal-specific modes addressed; effective dates checked so an amendment not yet in force is flagged. |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — no advice, no comparative regimes, no unquoted debate summaries, no document generation, refuse privileged content. |
| 9 | Escalation Logic       | ✅     | Specific deflect sentence. |
| 10 | Trust Surface         | ✅     | No hooks; MCP declared; no off-skill file writes. Exchange rules from a site that forbids republication are marked reference-only. |
| 11 | Freshness             | ✅     | No bundled references; text fetched live (national database, local systems, Legislative Yuan). |
| 12 | Schema                | ✅     | Frontmatter complete (description 913 chars); seven-step workflow; worked example (民法 §184); scope/limitations; confidence bands; failure modes; tools-used list. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills. |

**Legal failure mode check**:
- Legal advice vs. legal support: ✅ structural — verbatim article text + "not legal advice" notice.
- Privilege implications: ✅ explicit — refuses privileged content; output is public statute text.
- Accountability gap: ✅ explicit — output is statute text, not a concluded legal answer.

**Verdict**: **READY**

---

### `interpretation-lookup`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Same audience; states the weight of each source (釋字, 函釋, 決議, 判例 after the 2019 reform). |
| 2 | Work Shape             | ✅     | Pattern-Matched Review — lookup across constitutional, agency, court and review-body authority. |
| 3 | Delegation Threshold   | ✅     | Weighing which interpretation controls is the lawyer's call — stated in scope and deflect sentence. |
| 4 | Input Requirements     | ✅     | Routes by intent (number, topic, agency, 字號); does not apply the judgment date window to old 決議 / 判例; profile fallback. |
| 5 | Versioning / Ownership | ✅     | Plugin version + maintainer line. |
| 6 | Confidence Bands       | ✅     | High / Medium / Low explicit; Low says which sources were searched and never infers that nothing exists. |
| 7 | Failure Modes          | ✅     | Stale authority handled explicitly: every 函釋 citation carries the site's validity marking (停止適用 / 部分停止適用 / 適用中 / unmarked), and unmarked letters are never described as in force. |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — no advice, no judgments or statute text, no document generation, refuse privileged content. |
| 9 | Escalation Logic       | ✅     | Specific deflect sentence. |
| 10 | Trust Surface         | ✅     | No hooks; MCP declared; no off-skill file writes. Decisions are returned as officially published, including names some sites leave unmasked — stated in the skill. |
| 11 | Freshness             | ✅     | 釋字 / 憲判字 come from a bundled copy; rulings after the bundle are fetched live. Everything else is fetched live. |
| 12 | Schema                | ✅     | Frontmatter complete (description 843 chars); seven-step workflow; worked example with tool calls and output shape; confidence bands; source and limits; failure modes; tools-used list. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills. |

**Legal failure mode check**:
- Legal advice vs. legal support: ✅ structural — verbatim quotes, status notes and a "not legal advice" notice.
- Privilege implications: ✅ explicit — refuses privileged content.
- Accountability gap: ✅ explicit — outputs are research artifacts.

**Verdict**: **READY**

---

### `research-materials`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Professors, researchers, judges and clerks, in-house counsel, attorneys, students. |
| 2 | Work Shape             | ✅     | Pattern-Matched Review — literature and statistics lookup. |
| 3 | Delegation Threshold   | ✅     | Applying statistics to a client's facts is the lawyer's call — stated. |
| 4 | Input Requirements     | ✅     | Routes by intent (literature, statistics, sentencing); ROC-year handling stated; profile used for citation style only. |
| 5 | Versioning / Ownership | ✅     | Plugin version + maintainer line. |
| 6 | Confidence Bands       | ✅     | High / Medium / Low explicit; small sentencing samples marked Medium. |
| 7 | Failure Modes          | ✅     | Sentencing figures always carry the "not a sentencing guideline" note; NCL-licensed full text is never cached and must not be redistributed. |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — no paid databases, no outcome prediction, no document generation, refuse privileged content. |
| 9 | Escalation Logic       | ✅     | Specific deflect sentence. |
| 10 | Trust Surface         | ✅     | No hooks; MCP declared; no off-skill file writes. |
| 11 | Freshness             | ✅     | No bundled references; data fetched live. |
| 12 | Schema                | ✅     | Frontmatter complete (description 696 chars); five-step workflow; worked example; confidence bands; source and limits; failure modes; tools-used list. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills. |

**Legal failure mode check**:
- Legal advice vs. legal support: ✅ structural — figures and quotes attributed; sentencing answers carry the not-a-guideline note.
- Privilege implications: ✅ explicit — refuses privileged content.
- Accountability gap: ✅ explicit — outputs are research artifacts.

**Verdict**: **READY**

---

## Bottom line

All five v0.7 skills score **READY** against the Legal Skill Design Framework's 13 design parameters and three legal-specific failure modes. The skills themselves grant no tools and write only the practice profile; all data access goes through the bundled `mcp-taiwan-legal-db` server, which is a live scraper of official sites running on the user's machine (see its Disclaimer for how it handles robots.txt, browser checks and captchas, and what remains the user's responsibility).

This document reflects the author's pre-publication self-check. End users installing the plugin via `/legal-builder-hub:skill-installer` will receive an independent run of `/legal-builder-hub:skills-qa` as part of the install flow; that run is the authoritative QA, and any discrepancy with this report should be treated as the installer's call.
