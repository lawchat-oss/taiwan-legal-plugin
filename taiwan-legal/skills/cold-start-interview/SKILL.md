---
name: cold-start-interview
description: >
  One-time onboarding that captures the user's research preferences for the
  taiwan-legal plugin — preferred court levels and divisions to focus on,
  default date windows, citation style, abolished-regulation handling,
  and any standing subject scope. Writes a practice profile that
  judgment-search and statute-lookup read for their defaults. Run once
  when first installing the plugin, or any time the user wants to update
  defaults. Idempotent — safe to re-run.
argument-hint: "(no args — fully interactive)"
---

# /taiwan-legal:cold-start-interview

Captures the user's research defaults and writes them to a practice profile.

## Audience

The same audience as `judgment-search` and `statute-lookup` — Taiwan-legal researchers, attorneys, paralegals, law students. This is configuration only; no legal interpretation occurs here.

## Output file

Writes `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md` with this structure:

```markdown
# taiwan-legal practice profile

## Judgment search defaults
- court_levels: [<list>]
- divisions: [<list>]
- default_date_window: <string>

## Statute lookup defaults
- exclude_abolished: <true | false>

## Citation style
- style: <string>

## Standing scope
- subjects: <free text>
```

## Instructions

Ask the user the following, one at a time. After collecting answers, write the profile file. If the file already exists, ask whether to merge or replace before overwriting.

1. **Court levels** — Which Taiwan court levels do you most often research?
   - Common picks: 最高法院 / 高等法院 / 地方法院 / 行政法院 / 智慧財產及商業法院
   - Default if skipped: 最高法院、高等法院、地方法院

2. **Divisions** — Civil (民事), criminal (刑事), administrative (行政) — which do you focus on?
   - Default if skipped: all three

3. **Date window** — Default search range when the user does not specify? (e.g., "last 5 years", "last 10 years", "no default")
   - Default if skipped: last 5 years

4. **Citation style** — How should the plugin cite cases and statutes in summaries?
   - Options: 完整版 (字號 + 法院 + 日期 + URL) / 簡潔版 (字號 + 條號) / Bluebook-style with Taiwan adaptations
   - Default if skipped: 完整版

5. **Abolished regulations** — When searching regulations, exclude abolished ones by default?
   - Default if skipped: true

6. **Standing scope** — Any standing subject focus that should be remembered for every query? (Optional; leave blank for none.)

## Worked example

**Input** (user runs the skill):
> /taiwan-legal:cold-start-interview

**Interaction**:
> Q1: Which Taiwan court levels do you most often research? (default: 最高法院、高等法院、地方法院)
> User: 最高法院、智慧財產及商業法院
> Q2: Civil / criminal / administrative — which? (default: all three)
> User: 民事、行政
> Q3: Default date window? (default: last 5 years)
> User: last 10 years
> Q4: Citation style? (default: 完整版)
> User: <enter>
> Q5: Exclude abolished regulations by default? (default: true)
> User: <enter>
> Q6: Standing subject focus? (optional)
> User: 智慧財產

**Output file** (written to `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md`):
```markdown
# taiwan-legal practice profile

## Judgment search defaults
- court_levels: [最高法院, 智慧財產及商業法院]
- divisions: [民事, 行政]
- default_date_window: "last 10 years"

## Statute lookup defaults
- exclude_abolished: true

## Citation style
- style: "完整版 (字號 + 法院 + 日期 + URL)"

## Standing scope
- subjects: "智慧財產"
```

Followed by:
> Profile written. /taiwan-legal:judgment-search and /taiwan-legal:statute-lookup will use these defaults.

## Confidence bands

Not applicable — interactive configuration, not retrieval.

## Idempotency

Safe to re-run. Before overwriting an existing profile, ask the user whether to merge field-by-field or replace the whole file.

## What this skill does NOT do

- **Make legal judgments or recommendations.** Configuration only.
- **Write to anywhere outside** `~/.claude/plugins/config/taiwan-legal-plugin/taiwan-legal/CLAUDE.md`. Will not modify other CLAUDE.md files, shell configs, or settings.
- **Persist on remote servers.** Profile lives only in the user's local Claude Code config directory.

## Failure modes

- **File-write failure** (permission denied / disk full): surface the error, do not proceed silently.
- **Merge ambiguity** on re-run: ask the user; never silently overwrite.
- **Legal failure modes**: not applicable — this skill performs no legal work.

## Versioning

Plugin version: see `taiwan-legal/.claude-plugin/plugin.json`. Maintained by [lawchat-oss](https://github.com/lawchat-oss); issues and PRs at [github.com/lawchat-oss/taiwan-legal-plugin](https://github.com/lawchat-oss/taiwan-legal-plugin).
