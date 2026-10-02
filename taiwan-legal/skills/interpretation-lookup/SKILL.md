---
name: interpretation-lookup
description: >
  Look up Taiwanese interpretive authority other than court judgments and
  statute text: Grand Justices interpretations (釋字) and Constitutional
  Court judgments (憲判字) with their citation graph; ministries'
  administrative interpretations (行政函釋, 解釋令) from the Ministry of
  Justice, Labor, Health and Welfare, Finance, Economic Affairs, Interior,
  the Public Construction Commission and the Executive Yuan Gazette; and
  Supreme Court resolutions (決議), court legal Q&A conferences
  (法律問題座談), discontinued precedents (停止適用判例) and Judicial Yuan
  院字 / 院解字 interpretations. Use when the user asks how an agency
  interprets a provision, whether a 決議 or 判例 on a point exists, or what
  a 釋字 / 憲判字 held. Returns text pulled live from the official sources
  (釋字 / 憲判字 from a bundled copy of the official site).
argument-hint: "[keyword | agency | 字號 | 釋字 number]"
---

# /taiwan-legal:interpretation-lookup

Finds and retrieves interpretive authority: constitutional interpretations, agency interpretations, and court resolutions / Q&A conferences / precedents.

## Audience

Legal researchers, attorneys, paralegals, in-house counsel, and law students working in or with the Taiwanese legal system. Assumes familiarity with the weight of each source (釋字 binding on all organs; 函釋 binding on the agency but reviewable by courts; 決議 and 判例 lost their special binding force in the 2019 Grand Chamber reform). Not designed for end-consumers seeking legal advice.

## Instructions

1. **Load practice profile.** Read `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` for the user's citation style. If the file does not exist, say: "Run `/taiwan-legal:cold-start-interview` first to set defaults — or shall I proceed with no defaults?" Do **not** apply the profile's judgment-search date window here: 決議 predate the 2019 reform and 判例 / 院字・院解字 are decades old, so a "last 5 years" window would exclude them.

2. **Scope check.** Confirm the request is a lookup or research query, not an advice request (see "What this skill does NOT do" below).

3. **Identify intent and pick the tool.**
   - 釋字 / 憲判字 by number → `get_interpretation(case_id=...)`; add `reasoning_keyword=` or `include_reasoning=true` for the reasoning, `include_opinions=true` for Justices' opinions.
   - Constitutional topic → `search_interpretations(keyword=...)`; what a ruling cited → `get_citations(case_id=...)`.
   - How an agency reads a provision / 函釋 on a topic → `search_agency_interpretations(keyword=..., agency=...)`. Leave `agency` empty to search every source; use the agency's name (e.g. 勞動部, 財政部, 金管會) when the user names one. A known 字號 → `doc_number=...`. Then `get_agency_interpretation(interpretation_id=...)` for the full text.
   - 決議 / 法律問題座談 / 判例 / 院字・院解字 / 大法庭 → `search_precedents(keyword=..., category=...)`, then `get_precedent(precedent_id=...)`.
   - Years are ROC years (民國; 2026 = 115).

4. **Execute the MCP tool call(s).** Search without a date range unless the user asks for one.

5. **Check status before citing.** Read the editor's notes (`notes` on 函釋, `fields` → 編註 on 決議 / 判例) and report any 停止適用 / 不再援用 / 廢止 status next to the citation. Since the 2019 reform (法院組織法 §57-1): results in the 停止適用判例 category are discontinued — cite them only as historical material; 判例 that were *not* discontinued now carry only the weight of an ordinary Supreme Court decision; 決議 no longer bind and were in part expressly declared 不再援用.

6. **Present results.**
   - Search results: table of date, issuing agency or court, 字號, 要旨 / 主旨, id. Note any source listed with an `error` in `categories` as not searched.
   - Full text: quote the operative passage verbatim with 字號, date and source URL. Do not summarize without being asked.

7. **Cite faithfully.** Every cited interpretation must include its 字號 (or 會議次別), date and source URL.

## Worked example

**Input** (user):
> 勞動部對「加班費」有哪些函釋？

**Tool calls**:
1. `search_agency_interpretations(keyword="加班費", agency="勞動部")`
2. `get_agency_interpretation(interpretation_id="mol:e:勞動條 3:1100130312")` for the one the user picks

**Expected output shape**:
```
| 日期       | 機關   | 字號                              | 要旨                     |
|------------|--------|-----------------------------------|--------------------------|
| 2021-05-17 | 勞動部 | 勞動條 3字第 1100130312 號函       | 因疫情配合民生物資需求…   |
```

Followed by:
> Confidence: high (official source, exact keyword match). This is not legal advice; check the editor's notes for later changes before relying on any interpretation.

## Confidence bands

- **High**: exact 字號 / 釋字 number retrieved cleanly, or a keyword search whose hits come straight from the issuing agency's system.
- **Medium**: hits only from the Executive Yuan Gazette or the Judicial Yuan's cross-agency collection (coverage of individual 函 replies is partial there); or a source in `categories` returned an `error`.
- **Low**: zero results, all sources failed, or the user's agency has no dedicated system. **Do not infer** that no interpretation exists — say which sources were searched.

## Source and limits

- 釋字 / 憲判字: bundled copy of cons.judicial.gov.tw; rulings issued after the bundle are fetched live with links to their opinion PDFs.
- 函釋: queried live from each agency's own system (mojlaw.moj.gov.tw, laws.mol.gov.tw, mohwlaw.mohw.gov.tw, planpe.pcc.gov.tw, ttc.mof.gov.tw, gcis.nat.gov.tw, www.tipo.gov.tw, www.ris.gov.tw, www.nlma.gov.tw), the Judicial Yuan's FINT database, and gazette.nat.gov.tw. Agencies without their own system are covered only through interpretive rules published in the Gazette.
- 決議 / 座談 / 判例 / 院字・院解字: legal.judicial.gov.tw (FINT); at most the first 500 hits per category.

## What this skill does NOT do

- **Provide legal advice or decide which interpretation controls.** It retrieves what agencies and courts have said; weighing them is the lawyer's call.
- **Search court judgments or statute text.** Use `/taiwan-legal:judgment-search` and `/taiwan-legal:statute-lookup`.
- **Generate legal documents** (briefs, opinions). Out of scope.
- **Operate on inputs that imply privileged communications.** If the user pastes content that looks like attorney work product or client communications, refuse and remind the user this skill only takes public-record queries.

If a user request falls into any of the above, deflect with: "I can show you how agencies and courts have interpreted this — applying it to your facts is the lawyer's call."

## Failure modes

- **Legal advice vs. legal support**: addressed structurally — verbatim quotes, status notes, and an explicit "no legal advice" notice on every response.
- **Stale authority**: addressed by step 5 — every citation carries its 停止適用 / 不再援用 status when the source records one.
- **Accountability gap**: the lawyer remains the decision-maker — outputs are research artifacts, not concluded answers.

## Tools used

From the `taiwan-legal-db` MCP server (bundled with this plugin): `get_interpretation`, `search_interpretations`, `get_citations`, `search_agency_interpretations`, `get_agency_interpretation`, `search_precedents`, `get_precedent`.

## Versioning

Plugin version: see `taiwan-legal/.claude-plugin/plugin.json`. Maintained by [lawchat-oss](https://github.com/lawchat-oss); issues and PRs at [github.com/lawchat-oss/taiwan-legal-plugin](https://github.com/lawchat-oss/taiwan-legal-plugin).
