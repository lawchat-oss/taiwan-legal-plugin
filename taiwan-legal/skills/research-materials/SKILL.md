---
name: research-materials
description: >
  Find Taiwanese legal research materials from official and open-access
  sources: Judicial Yuan research reports (司法研究年報 / 專題研究報告),
  the National Central Library's periodical index (臺灣期刊論文索引) with
  abstracts and authorised full text, government-funded research projects
  (GRB), open-access law journals (中研院法學期刊, 政大法學評論), official
  statistics (司法統計年報 / 月報, 法務統計, 犯罪狀況及其分析), and the
  Judicial Yuan's sentencing statistics (事實型量刑資訊系統). Use when the
  user wants scholarship or empirical data on a legal question — e.g. what
  has been written on a doctrine, how many cases of a kind courts closed,
  or the historical sentence range for an offence. Paid databases (月旦,
  華藝, 法源, Lawsnote) are out of scope.
argument-hint: "[topic | author | statistic | offence]"
---

# /taiwan-legal:research-materials

Retrieves scholarship and statistics on Taiwanese law from official and open-access sources.

## Audience

Law professors, legal researchers, judges and their clerks, in-house counsel, attorneys, and students who need secondary sources or empirical data. Not designed for end-consumers seeking legal advice.

## Instructions

1. **Load practice profile.** Read `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` for the user's citation style. If the file does not exist, proceed without defaults. Do not apply the profile's judgment date window to literature or statistics.

2. **Scope check.** Confirm the request is a research query, not an advice request (see "What this skill does NOT do").

3. **Identify intent and pick the tool.**
   - Articles, reports, theses on a topic or by an author → `search_legal_literature(keyword=..., source=...)`; leave `source` empty to search every source, or name one (司法研究年報, 期刊, GRB, 開放期刊). Then `get_legal_literature(literature_id=...)` for the abstract and, where published openly, the full text.
   - Court caseload / case outcomes / prosecution and conviction figures → `search_statistics(keyword=..., source=..., year=...)` with a term from the table title (收結, 上訴, 民事, 有罪 …), then `get_statistics(statistics_id=...)` for the table (pipe-separated rows) or report text.
   - Historical sentence ranges → `get_sentencing_statistics(crime=...)` first to see the available `law_options` and `factor_options`, then narrow with `law=`, `court=`, `factors="累犯=是"`, `year_from=`, `year_to=`. Only ten offence types are covered.
   - Years are ROC years (民國; 2026 = 115).

4. **Present results.**
   - Literature: author, title, venue (journal + volume/issue, or 司法研究年報第 N 輯), year, id. Mark which items have full text.
   - Statistics: quote the figures with the table title, period and source URL; do not extrapolate beyond the table.
   - Sentencing: report the count of judgments matched, the average / minimum / maximum per penalty type, and the filters used. Always add the tool's `note`: this describes past judgments in the system's sample, not a sentencing guideline.

5. **Cite faithfully.** Literature: 作者，〈篇名〉，《刊名》，卷期，頁碼，年份 (or the report's 輯 / 篇). Statistics: agency, table title, period, URL.

## Worked example

**Input** (user):
> 竊盜罪累犯在臺北地院大概會被判多久？

**Tool calls**:
1. `get_sentencing_statistics(crime="竊盜")` → see law and factor options
2. `get_sentencing_statistics(crime="竊盜", law="第320條第1項", court="臺北地院", factors="累犯=是", year_from=110, year_to=113)`

**Expected output shape**:
> 司法院事實型量刑資訊系統，竊盜（刑法第 320 條第 1 項）、臺北地院、累犯，110–113 年：符合 521 件。有期徒刑 265 件，平均 3.8 月（2 月至 3 年）；拘役 228 件，平均 32 日；罰金 28 件。
> 這是過去判決的統計分布，不是量刑基準，也不構成法律意見。

## Confidence bands

- **High**: an exact title / author / table match, or sentencing statistics with an explicit filter set and a non-trivial sample.
- **Medium**: broad keyword hits that need user filtering; literature with abstract only; sentencing samples under ~30 judgments.
- **Low**: zero results or a source error. **Do not infer** that nothing has been written or decided — say which sources were searched.

## Source and limits

- Literature: jirs.judicial.gov.tw, tpl.ncl.edu.tw, GRB, and the journals' own sites. The National Central Library's licensed full text is for personal reading only — do not store or redistribute it. GRB final reports and the NDLTD thesis system require a captcha and are not fetched; the tool returns a link instead.
- Statistics: www.judicial.gov.tw, rjsd.moj.gov.tw, www.cprc.moj.gov.tw. Tables are converted to text as published.
- Sentencing: intellisen.judicial.gov.tw aggregate statistics only (the per-judgment list is reserved for the court's internal users and is not used).

## What this skill does NOT do

- **Access paid databases** (月旦, 華藝, 法源, Lawsnote) or bypass captchas.
- **Predict a sentence or outcome.** Statistics describe past cases; applying them to a client's facts is the lawyer's call.
- **Generate legal documents.** Out of scope.
- **Operate on inputs that imply privileged communications.** If the user pastes content that looks like attorney work product or client communications, refuse and remind the user this skill only takes public-record queries.

If a user request falls into any of the above, deflect with: "I can show you the published research and statistics — applying them to your case is the lawyer's call."

## Failure modes

- **Legal advice vs. legal support**: addressed structurally — figures and quotes are attributed, and every sentencing answer carries the not-a-guideline note.
- **Licence misuse**: NCL full text is never cached by the server and must not be redistributed.
- **Accountability gap**: the lawyer or researcher remains the decision-maker — outputs are research artifacts.

## Tools used

From the `taiwan-legal-db` MCP server (bundled with this plugin): `search_legal_literature`, `get_legal_literature`, `search_statistics`, `get_statistics`, `get_sentencing_statistics`.

## Versioning

Plugin version: see `taiwan-legal/.claude-plugin/plugin.json`. Maintained by [lawchat-oss](https://github.com/lawchat-oss); issues and PRs at [github.com/lawchat-oss/taiwan-legal-plugin](https://github.com/lawchat-oss/taiwan-legal-plugin).
