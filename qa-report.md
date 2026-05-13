# Skills QA — self-evaluation report

> Author self-evaluation against the Legal Skill Design Framework v0.1 (13 design parameters, 3 legal-specific failure modes, 3-band verdict) as published in [`anthropics/claude-for-legal`](https://github.com/anthropics/claude-for-legal) `legal-builder-hub/skills/skills-qa/SKILL.md`. **End users should run `/legal-builder-hub:skills-qa` against the installed skill directory for an independent assessment** — this document is the author's pre-publication check, not a substitute for runtime QA.

Evaluated: **2026-05-13**
Source: first-party (LawChat OSS)
Skills evaluated: `cold-start-interview`, `judgment-search`, `statute-lookup` (in `taiwan-legal/skills/`)

---

## Prompt-injection heuristic scan

Author self-scan against the 10 categories in the QA spec (override/ignore instructions, authority claims, config-override, out-of-scope reads, out-of-scope writes, external URLs, hidden content, shell/code execution, credential-adjacent asks, legal authority overclaiming).

**Findings**: none detected across all three SKILL.md files. The only file writes are to `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` (in-scope for plugin config); no hooks, no `Bash` / `WebFetch` / `WebSearch` tool grants, no encoded blobs, no zero-width characters or HTML directives. External URLs are limited to documented public-record portals (`judgment.judicial.gov.tw`, `law.moj.gov.tw`, `cons.judicial.gov.tw`) reached only via the declared `taiwan-legal-db` MCP server.

> This is a heuristic scan by the author, not a security audit. End users should rely on `/legal-builder-hub:skill-installer`'s independent scan and human approval gate.

---

## Dependency map

| Direction | What | Notes |
|---|---|---|
| Upstream | `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` | Practice profile written by `cold-start-interview`; read by the other two skills. Skills handle absence by prompting the user to run cold-start. |
| Upstream | `taiwan-legal-db` MCP server (bundled via `.mcp.json`) | stdio transport; depends on `pip install mcp-taiwan-legal-db` being available on the user's PATH. |
| Downstream | `cold-start-interview` writes profile; no other writes | No skill writes outside `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/`. |
| Auto-triggers | none | No `hooks/hooks.json`; no agents; no scheduled invocations. |
| Breakage risk | If MCP server unavailable, all three skills surface a clear error and stop. | No silent fallback. |

---

## Parameter evaluation

### `cold-start-interview`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Stated: same audience as the lookup skills; config-only. |
| 2 | Work Shape             | ✅     | N/A — interactive configuration, not legal work; noted explicitly. |
| 3 | Delegation Threshold   | ✅     | N/A — no legal interpretation occurs. |
| 4 | Input Requirements     | ✅     | Defaults provided for every question; user may skip any. |
| 5 | Versioning / Ownership | ✅     | Plugin `version: 0.1.0`; maintainer line in skill. |
| 6 | Confidence Bands       | ✅     | N/A — interactive configuration; declared. |
| 7 | Failure Modes          | ✅     | File-write failure handled; merge-ambiguity handled; legal failure modes N/A and declared. |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — config-only, single-file write, no remote persistence. |
| 9 | Escalation Logic       | ✅     | N/A — no high-stakes legal work. |
| 10 | Trust Surface         | ✅     | Writes only `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md`; no hooks; MCP declared with source attribution; no `Bash` / `WebFetch`. |
| 11 | Freshness             | ✅     | N/A — no bundled `references/` content. |
| 12 | Schema                | ✅     | Frontmatter complete; workflow section; worked example present; scope/limitations section; idempotency section. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills (US-focused). |

**Legal failure mode check**:
- Legal advice vs. legal support: ✅ N/A (configuration only) — declared.
- Privilege implications: ✅ N/A — no work product.
- Accountability gap: ✅ N/A — no judgment.

**Verdict**: **READY**

---

### `judgment-search`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Legal researchers, attorneys, paralegals, in-house counsel, law students working in/with the Taiwanese legal system. |
| 2 | Work Shape             | ✅     | Pattern-Matched Review — research/lookup with explicit non-summarization rule. |
| 3 | Delegation Threshold   | ✅     | "Conclusions, opinions, and recommendations are the user's (or their attorney's) responsibility" — structural via verbatim quoting rule. |
| 4 | Input Requirements     | ✅     | Ambiguity handling (search vs. read full text → ask); profile fallback prompts cold-start. |
| 5 | Versioning / Ownership | ✅     | Plugin version + maintainer line. |
| 6 | Confidence Bands       | ✅     | High / Medium / Low explicit, with "do not synthesize" rule on Low. |
| 7 | Failure Modes          | ✅     | All three legal-specific modes addressed (see below); general modes covered in "Source and limits". |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — no advice, no summary without quote, no jurisdiction comparison, no document generation, refuse privileged content. |
| 9 | Escalation Logic       | ✅     | Specific deflect sentence: "I can show you what the courts have said on this question — would that help, or do you need an attorney's analysis?" |
| 10 | Trust Surface         | ✅     | No hooks; MCP declared (`taiwan-legal-db` stdio, source attributed); no off-skill file writes; no overclaiming. |
| 11 | Freshness             | ✅     | No bundled `references/` content; data fetched live from `judgment.judicial.gov.tw` at query time. |
| 12 | Schema                | ✅     | Frontmatter complete (≤1024 char description: 616 chars); workflow with 7 numbered steps; worked example with concrete tool calls and output shape; scope/limitations section; confidence bands; failure modes; tools-used list. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills (US-focused). |

**Legal failure mode check**:
- Legal advice vs. legal support: ✅ structural — verbatim quoting + explicit "no legal advice" notice on every response.
- Privilege implications: ✅ explicit — refuses to process inputs that look like privileged communications.
- Accountability gap: ✅ explicit — lawyer remains the decision-maker; output is research artifact (search hits + quoted text), not a concluded answer.

**Verdict**: **READY**

---

### `statute-lookup`

| # | Parameter              | Status | Notes |
|---|------------------------|--------|-------|
| 1 | Audience               | ✅     | Same audience as `judgment-search`; familiarity with Taiwanese statute structure (法 / 條 / 項 / 款 / 目). |
| 2 | Work Shape             | ✅     | Pattern-Matched Review — canonical lookup. |
| 3 | Delegation Threshold   | ✅     | "Interpretation, application to facts, and operative legal conclusions are the user's (or their attorney's) responsibility." |
| 4 | Input Requirements     | ✅     | Ambiguity handling (multiple regulation matches → disambiguation prompt); profile fallback. |
| 5 | Versioning / Ownership | ✅     | Plugin version + maintainer line. |
| 6 | Confidence Bands       | ✅     | High / Medium / Low explicit; Low includes "do not invent or guess article content". |
| 7 | Failure Modes          | ✅     | All three legal-specific modes addressed (see below); coverage limits of subsidiary rules (函釋 / 解釋令) declared. |
| 8 | Scope Boundaries       | ✅     | Explicit "What this skill does NOT do" — no advice, no comparative regimes, no deep legislative history, no document generation, refuse privileged content. |
| 9 | Escalation Logic       | ✅     | Specific deflect sentence: "I can show you the statute text — interpretation is the lawyer's call." |
| 10 | Trust Surface         | ✅     | No hooks; MCP declared; no off-skill file writes; no overclaiming. |
| 11 | Freshness             | ✅     | No bundled `references/` content; data fetched live from `law.moj.gov.tw` at query time. |
| 12 | Schema                | ✅     | Frontmatter complete (≤1024 char description: 601 chars); workflow with 7 numbered steps; worked example (民法 §184 walkthrough); scope/limitations; confidence bands; failure modes; tools-used list. |
| 13 | Conflicts             | ✅     | No overlap with existing `claude-for-legal` skills. |

**Legal failure mode check**:
- Legal advice vs. legal support: ✅ structural — verbatim article text + explicit "no legal advice" notice.
- Privilege implications: ✅ explicit — refuses privileged content; output is public statute text.
- Accountability gap: ✅ explicit — lawyer remains decision-maker; output is statute text, not a concluded legal answer.

**Verdict**: **READY**

---

## Bottom line

All three v0.1 skills score **READY** against the Legal Skill Design Framework's 13 design parameters and three legal-specific failure modes. The plugin operates as a thin access layer over `mcp-taiwan-legal-db`, with no autonomous behavior (no hooks, no scheduled agents), no bundled mutable reference content, and structural guardrails that route legal judgment back to the lawyer.

This document reflects the author's pre-publication self-check. End users installing the plugin via `/legal-builder-hub:skill-installer` will receive an independent run of `/legal-builder-hub:skills-qa` as part of the install flow; that run is the authoritative QA, and any discrepancy with this report should be treated as the installer's call.
