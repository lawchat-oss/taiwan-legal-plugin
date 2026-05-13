---
name: judgment-search
description: >
  Search Taiwan court judgments published on the Judicial Yuan portal
  (judgment.judicial.gov.tw) and retrieve full judgment text. Use when the
  user asks to find Taiwanese court rulings by keyword, court level, division,
  date range, or case number; to read the full reasoning of a specific
  judgment by its 字號 (case ID); or to summarize what Taiwan courts have
  decided on a given topic. Returns structured search results and full-text
  bodies pulled live from the official source. This is an access layer —
  judgments come unmodified from judicial.gov.tw and reflect that source's
  coverage and freshness.
argument-hint: "[keyword | court | 字號 | date range]"
---

# /taiwan-legal:judgment-search

Searches Taiwan's open judgment database and returns structured results or full text.

## Audience

Legal researchers, attorneys, paralegals, in-house counsel, and law students working in or with the Taiwanese legal system. Assumes familiarity with Taiwan court structure (最高法院 / 高等法院 / 地方法院 / 行政法院 / 智慧財產及商業法院) and the 字號 case-ID convention. Not designed for end-consumers seeking legal advice.

## Instructions

1. **Load practice profile.** Read `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` for the user's default court levels, date windows, and citation style. If the file does not exist, say: "Run `/taiwan-legal:cold-start-interview` first to set defaults — or shall I proceed with no defaults?"

2. **Scope check.** Before running the tool, confirm the request falls inside scope (see "What this skill does NOT do" below). If the user is asking for a legal opinion, prediction, or recommendation on a case → do **not** silently produce one; surface the limitation and offer a research-only path instead.

3. **Identify intent from the user's query.**
   - "search" / keyword / topic → use `search_judgments(keyword, court, date_range, ...)` from the `taiwan-legal-db` MCP server.
   - 字號 given / "read full text" → use `get_judgment(jid=...)`.
   - URL given → use `get_judgment(url=...)`.
   - Ambiguous → ask which the user wants.

4. **Execute the MCP tool call** with profile defaults applied only when the user did not specify (e.g., default court level, default date range). Explicit user-provided values override defaults.

5. **Apply confidence bands** (see below) when characterizing results.

6. **Present results.**
   - Search results: tabular — case ID (字號), court, division (民/刑/行), date, cause (案由), URL.
   - Full text: render the structured fields (主文, 事實, 理由, 引用法條) without summarization unless the user asks. For excerpts, quote verbatim with the citation attached.

7. **Cite faithfully.** Every claim attributed to a judgment must include 字號 + 法院 + 日期 + URL. Do not paraphrase legal holdings without quoting the operative text.

## Worked example

**Input** (user):
> 找最高法院近五年關於「不當得利返還請求權消滅時效」的民事判決

**Plan**: search across 最高法院 民事 division, keyword「不當得利返還請求權 消滅時效」, last 5 years.

**Tool call**: `search_judgments(keyword="不當得利返還請求權 消滅時效", court="最高法院", date_range="last_5_years")`

**Expected output shape**:
```
| 字號                    | 法院     | 日期       | 案由         | URL  |
|------------------------|----------|------------|--------------|------|
| 109 年度台上字第 1234 號 | 最高法院 | 2020-08-12 | 返還不當得利 | …    |
| 108 年度台上字第 567 號  | 最高法院 | 2019-11-03 | 給付         | …    |
```

Followed by:
> Found N matches (confidence: high — exact-match search). To read full reasoning on any case, ask "讀 109 年度台上字第 1234 號 全文" or paste the URL.
> This is not legal advice; results are research material.

## Confidence bands

- **High**: exact case ID supplied and retrieved cleanly; specific keyword search returning ≥1 result on the requested court/date range.
- **Medium**: keyword search returning broad results that may require user filtering; case ID retrieved but with parsing warnings.
- **Low**: zero results, WAF / network failure after retry, or ambiguous query that could match multiple interpretations. **Do not synthesize** — surface the gap to the user and ask how to refine.

## Source and limits

- Data origin: judgment.judicial.gov.tw (Judicial Yuan open-data portal). Coverage equals what that portal publishes; older or unpublished judgments will not appear.
- Some queries trigger an F5 WAF on the upstream source; the bundled MCP server uses a hybrid httpx + Playwright strategy to handle this, but transient failures can still occur — retry once before reporting an error.

## What this skill does NOT do

- **Provide legal advice or predictions.** This skill retrieves and presents what courts have said. Conclusions, opinions, and recommendations are the user's (or their attorney's) responsibility.
- **Summarize legal holdings without quoting the operative text.** Summaries on legal questions risk distortion; the skill quotes 主文 / 理由 verbatim with citation.
- **Compare jurisdictions** (Taiwan vs. other ROC-adjacent or common-law systems). Out of scope — escalate to a comparative-law skill if needed.
- **Generate legal documents** (briefs, motions, opinions). Out of scope.
- **Operate on inputs that imply privileged communications.** If the user pastes content that looks like attorney work product or client communications, refuse to process it and remind the user this skill only takes public-record queries.

If a user request falls into any of the above, deflect with: "I can show you what the courts have said on this question — would that help, or do you need an attorney's analysis?"

## Failure modes

- **Legal advice vs. legal support**: addressed structurally — verbatim quoting + explicit "no legal advice" notice on every response.
- **Privilege**: this skill operates only on public records and never produces work product. Users should not paste privileged content into queries; if detected, refuse and warn.
- **Accountability gap**: the lawyer remains the decision-maker — outputs are research artifacts (search hits, quoted text), not concluded answers.

## Tools used

From the `taiwan-legal-db` MCP server (bundled with this plugin): `search_judgments`, `get_judgment`.

## Versioning

Plugin version: see `taiwan-legal/.claude-plugin/plugin.json`. Maintained by [lawchat-oss](https://github.com/lawchat-oss); issues and PRs at [github.com/lawchat-oss/taiwan-legal-plugin](https://github.com/lawchat-oss/taiwan-legal-plugin).
