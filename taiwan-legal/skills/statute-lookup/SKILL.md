---
name: statute-lookup
description: >
  Look up Taiwan statutes, regulations, and orders from the National
  Regulation Database (law.moj.gov.tw, ~11,700 regulations). Use when the
  user asks to retrieve the current text of a Taiwanese law by name
  (e.g., 民法, 公司法, 刑法); to find a specific article within a regulation;
  to search regulations by keyword across the full corpus; or to identify
  which regulations govern a subject area. Returns structured article-level
  data pulled live from the official source. This is an access layer —
  text comes unmodified from law.moj.gov.tw and reflects whatever version
  that source publishes.
argument-hint: "[regulation name | article number | keyword]"
---

# /taiwan-legal:statute-lookup

Queries Taiwan's National Regulation Database and returns full-text article data.

## Audience

Legal researchers, attorneys, paralegals, in-house counsel, and law students working in or with the Taiwanese legal system. Assumes familiarity with the Taiwanese statute structure (法 / 法律 / 命令; 條 / 項 / 款 / 目). Not designed for end-consumers seeking legal advice.

## Instructions

1. **Load practice profile.** Read `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` for the user's preferred citation style and abolished-regulation handling. If the file does not exist, say: "Run `/taiwan-legal:cold-start-interview` first to set defaults — or shall I proceed with no defaults?"

2. **Scope check.** Confirm the request is a lookup or research query, not an advice request (see "What this skill does NOT do" below).

3. **Identify intent.**
   - Specific regulation by name (e.g., "民法") → resolve via `get_pcode(law_name=...)`, then `query_regulation(pcode=...)`.
   - Keyword search across all regulations → `search_regulations(keyword=..., exclude_abolished=...)`.
   - Specific article (e.g., "民法 §184") → `query_regulation(law_name=..., article=...)`.

4. **Execute the MCP tool call.** Apply profile defaults when the user did not specify (e.g., `exclude_abolished=true` if set in the profile).

5. **Apply confidence bands** (see below).

6. **Present results.**
   - Search results: list regulation names with pcode, abolished status, and URL.
   - Article text: render verbatim with article number, body, and any subsection labels (項 / 款 / 目). Do not summarize without being asked.

7. **Cite faithfully.** Every cited article must include 法規全名 + 第 X 條 + URL.

## Worked example

**Input** (user):
> 民法 184 條 全文 + 引用

**Plan**: resolve 民法 → pcode B0000001, then query article 184.

**Tool calls**:
1. `get_pcode(law_name="民法")` → `B0000001`
2. `query_regulation(pcode="B0000001", article="184")`

**Expected output shape**:
```
民法 第 184 條（侵權行為）
第一項：因故意或過失，不法侵害他人之權利者，負損害賠償責任。故意以背於善良風俗之方法，加損害於他人者亦同。
第二項：違反保護他人之法律，致生損害於他人者，負賠償責任。但能證明其行為無過失者，不在此限。
出處：https://law.moj.gov.tw/LawClass/LawSingle.aspx?pcode=B0000001&flno=184
```

Followed by:
> Confidence: high (canonical lookup). This is not legal advice; the article text reflects the version published by law.moj.gov.tw at retrieval time.

## Confidence bands

- **High**: exact regulation + article retrieved cleanly from canonical pcode.
- **Medium**: keyword search returns multiple plausible matches; user disambiguation suggested. Regulation found but abolished — surface status explicitly.
- **Low**: zero results, pcode resolution failed, network failure after retry, or ambiguous regulation name (e.g., "勞動法" — there is no single statute by that name; surface the cluster of relevant statutes instead). **Do not invent or guess** article content.

## Source and limits

- Data origin: law.moj.gov.tw (Ministry of Justice national regulation portal). Coverage equals what that portal publishes.
- The portal reflects the version of each regulation as of its own last update; very recent amendments may take days to appear.
- Subsidiary rules (函釋, 解釋令) are partially covered by law.moj.gov.tw but not exhaustively; for definitive coverage of subsidiary rules consult the issuing agency directly.

## What this skill does NOT do

- **Provide legal advice or interpretation.** This skill returns the text of statutes. Interpretation, application to facts, and operative legal conclusions are the user's (or their attorney's) responsibility.
- **Compare statutory regimes across jurisdictions.** Taiwan-only; comparative work is out of scope.
- **Track legislative history with citations to legislative committee reports.** Basic 修法沿革 is available from the source portal, but deep legislative history research requires the Legislative Yuan database (out of scope for v0.1).
- **Generate legal documents** (contracts, opinions, briefs). Out of scope.
- **Operate on inputs that imply privileged communications.** If the user pastes content that looks like attorney work product or client communications, refuse and remind the user this skill only takes public statute queries.

If a user request falls into any of the above, deflect with: "I can show you the statute text — interpretation is the lawyer's call."

## Failure modes

- **Legal advice vs. legal support**: addressed structurally — verbatim article text + explicit "no legal advice" note on every response.
- **Privilege**: this skill operates only on public statute text and never produces work product. If user pastes privileged content, refuse and warn.
- **Accountability gap**: the lawyer remains the decision-maker — output is statute text, not a concluded legal answer.

## Tools used

From the `taiwan-legal-db` MCP server (bundled with this plugin): `query_regulation`, `get_pcode`, `search_regulations`.

## Versioning

Plugin version: see `taiwan-legal/.claude-plugin/plugin.json`. Maintained by [lawchat-oss](https://github.com/lawchat-oss); issues and PRs at [github.com/lawchat-oss/taiwan-legal-plugin](https://github.com/lawchat-oss/taiwan-legal-plugin).
