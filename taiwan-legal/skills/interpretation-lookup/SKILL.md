---
name: interpretation-lookup
description: >
  Look up Taiwanese interpretive authority other than judgments and
  statute text: 釋字 / 憲判字 with their citation graph, case files and
  pending docket; administrative interpretations (函釋, 解釋令) and
  examination guidelines from about thirty official systems; Supreme
  Court resolutions (決議), legal Q&A conferences, discontinued
  precedents, 院字 / 院解字 and curated judgments with 裁判要旨; and
  decisions of administrative-appeal and quasi-judicial bodies (訴願,
  FTC, procurement complaints, labour adjudication, civil-service
  protection, FSC sanctions, Control Yuan, lawyer discipline). Use when
  the user asks how an agency interprets a provision, whether a 決議 or
  判例 exists, what a 釋字 / 憲判字 held or which later rulings cited it,
  or how a review body decided a kind of case. Text is pulled live from
  official sources (釋字 / 憲判字 from a bundled copy).
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
   - Constitutional topic → `search_interpretations(keyword=...)`; what a ruling cited → `get_citations(case_id=...)`; which later 釋字 / 憲判字 cited it → `get_citations(case_id=..., direction="cited_by")`.
   - Case-file materials (聲請書, 答辯書, 鑑定意見, 法庭之友意見書, 言詞辯論筆錄, 爭點題綱) → `get_constitutional_case_file(case_id=...)`, optionally with `keyword=` to find which filings discuss a point; read one with `document_id=`. Cases not yet decided → `search_constitutional_docket(keyword=..., status="pending" | "hearing" | "amicus")`.
   - How an agency reads a provision / 函釋 on a topic → `search_agency_interpretations(keyword=..., agency=...)`. Leave `agency` empty to search every source; use the agency's name (e.g. 勞動部, 財政部, 金管會, 銓敘部, 人事總處, 消保處, 地政司, 臺北市, 新北市) when the user names one; 外交部, 退輔會, 核安會, 國發會, NCC, 客委會, 僑委會, 運動部, 關務署新頒釋函 and 陸委會主站廣告函釋 are searched only when named. A known 字號 → `doc_number=...`. Then `get_agency_interpretation(interpretation_id=...)` for the full text. Patent / trademark examination guidelines: `agency="智慧局"` with a chapter term such as 專利要件 or 混淆誤認.
   - 決議 / 法律問題座談 / 判例 / 院字・院解字 / 大法庭 → `search_precedents(keyword=..., category=...)`, then `get_precedent(precedent_id=...)`.
   - Curated judgments with 裁判要旨 → `search_precedents(keyword=..., category="精選裁判")`; only those a court designated 具參考價值 / 足資討論 → `category="具參考價值裁判"` (items carry `reference_value`).
   - Administrative appeal and quasi-judicial decisions → `search_administrative_decisions(keyword=..., source=...)`, then `get_administrative_decision(decision_id=...)`. Without `source` it searches 行政院訴願, 公平會, 不當勞動行為裁決, 保訓會, and 金管會裁罰. Name the source for 工程會採購申訴審議判斷 (`source="採購申訴"`; no keyword search upstream — use the case number such as 訴1130123 or a year), 監察院 (slow), 律師懲戒 (needs a precise keyword), 醫事懲戒 (currently posted medical disciplinary notices; matches names, locations and certificate numbers), or a ministry's / local government's 訴願 (e.g. `source="臺北市"`, or `source="訴願"` for all of them). Decisions come back as the official sites publish them; older Executive Yuan (filed up to 民國 108) and 法務部 decisions and 原民會 decisions show the parties' names. Executive Yuan cases filed up to 民國 108 use the 院臺訴字 number as their id (e.g. `ey:1070210137`).
   - Years are ROC years (民國; 2026 = 115).

4. **Execute the MCP tool call(s).** Search without a date range unless the user asks for one.

5. **Check status before citing.** For 函釋, read `status` / `status_note` on every result and full text: `停止適用` means the official site marks it discontinued — report it with the note, never cite it as current law. `部分停止適用` means only part of it was discontinued — read the full text and `status_note` to identify which part, and cite only the part still in force. `適用中` means the site lists it as current (or, for 財政部, it is in the latest 法令彙編). **No `status` means the site does not say** — never describe such a letter as in force; say its status is unmarked and point to the editor's notes. Read the editor's notes too (`notes` on 函釋, `fields` → 編註 on 決議 / 判例) and report any 停止適用 / 不再援用 / 廢止 status next to the citation. Since the 2019 reform (法院組織法 §57-1): results in the 停止適用判例 category are discontinued — cite them only as historical material; 判例 that were *not* discontinued now carry only the weight of an ordinary Supreme Court decision; 決議 no longer bind and were in part expressly declared 不再援用.

6. **Present results.**
   - Search results: table of date, issuing agency or court, 字號, 要旨 / 主旨, status (停止適用 / 部分停止適用 / 適用中 / 未標示), id. Note any source listed with an `error` in `categories` as not searched.
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
- 函釋: queried live from each agency's own system (45, listed in the server's SOURCES.md), the Judicial Yuan's FINT database, and gazette.nat.gov.tw. Agencies without their own system are covered only through interpretive rules published in the Gazette. Some agencies (金管會, 教育部 …) file interpretations among their administrative rules, so results mix in ordinary rules.
- 決議 / 座談 / 判例 / 院字・院解字 / 精選裁判: legal.judicial.gov.tw (FINT); at most the first 500 hits per category.
- Decisions: each review body's official site; scanned PDFs return only a link.

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

From the `taiwan-legal-db` MCP server (bundled with this plugin): `get_interpretation`, `search_interpretations`, `get_citations`, `search_constitutional_docket`, `get_constitutional_case_file`, `search_agency_interpretations`, `get_agency_interpretation`, `search_precedents`, `get_precedent`, `search_administrative_decisions`, `get_administrative_decision`.

## Versioning

Plugin version: see `taiwan-legal/.claude-plugin/plugin.json`. Maintained by [lawchat-oss](https://github.com/lawchat-oss); issues and PRs at [github.com/lawchat-oss/taiwan-legal-plugin](https://github.com/lawchat-oss/taiwan-legal-plugin).
