---
name: weekly-report
description: Generate a weekly engineering report from git commits. Use when the user asks to write a weekly report, summarize weekly work, or generate a work recap.
---

# Weekly Report Generator

## Purpose

Turn this week's meaningful git commit subjects into a concise weekly report, then append the final report to `week.log` in the project root.

## Trigger Phrases

Apply this skill when the user says things like:
- "write my weekly report"
- "generate weekly report"
- "summarize this week's work"
- "帮我写周报"
- "生成周报"
- "整理本周工作"

## Workflow

1. Identify the reporting range.
2. Collect commits authored by the current user.
3. Remove noisy or trivial commit subjects.
4. Merge duplicate or highly similar commit subjects.
5. Group items into:
   - Feature Development
   - Bug Fixes
   - Optimizations
6. Rewrite bullet points in business-value language.
7. Infer a draft "Next Week Plan".
8. Run a quality check before output.
9. Append one report block to `week.log` (do not overwrite).
10. Confirm completion and ask whether to refine "Next Week Plan".

## Date Range Rules

- Default: from this Monday to now.
- If today is Monday and the user asks for "last week", use previous Monday to previous Sunday.
- Date header format: `YYYY/MM/DD - YYYY/MM/DD`.
- Prefer explicit date strings over natural language dates for repeatability.

Recommended date generation (macOS):

```bash
START_DATE="$(date -v-monday +%Y/%m/%d)"
END_DATE="$(date +%Y/%m/%d)"
```

For last week on Monday (macOS):

```bash
START_DATE="$(date -v-2monday +%Y/%m/%d)"
END_DATE="$(date -v-sunday +%Y/%m/%d)"
```

Linux fallback (GNU date):

```bash
START_DATE="$(date -d 'monday this week' +%Y/%m/%d)"
END_DATE="$(date +%Y/%m/%d)"
```

If date commands fail, fallback to:

```bash
git log --since="last Monday" --until="now"
```

## Commit Collection

Prefer email first, then fallback to name:

```bash
AUTHOR_EMAIL="$(git config user.email)"
AUTHOR_NAME="$(git config user.name)"
```

Use one of these commands:

```bash
# Preferred: by email
git log --since="$START_DATE 00:00:00" --until="$END_DATE 23:59:59" --author="$AUTHOR_EMAIL" --pretty=format:"%s" --no-merges

# Fallback: by name
git log --since="$START_DATE 00:00:00" --until="$END_DATE 23:59:59" --author="$AUTHOR_NAME" --pretty=format:"%s" --no-merges
```

## Filtering Rules

Drop items that are clearly low-value noise, including:
- `wip`
- `typo`
- `merge branch`
- `revert`
- pure dependency bumps without feature impact
- empty or duplicate subjects

If all commits are filtered out, still generate a report with:
- a short status note
- a realistic "Next Week Plan" draft

## De-duplication Rules

- Remove exact duplicate subjects.
- Merge near-duplicate subjects that describe the same user-facing outcome.
- Prefer one stronger business-facing sentence over multiple technical micro-commits.
- If one item is a strict subset of another, keep the broader user-value item.

## Classification Rules

Classify by intent, not strict prefix only:

- Feature Development:
  - `feat`, `support`, `add`, `introduce`
- Bug Fixes:
  - `fix`, `hotfix`, `bug`, `resolve`
- Optimizations:
  - `refactor`, `perf`, `optimize`, `improve`, `cleanup`, `chore` (only if user-visible value exists)

If uncertain, classify based on user-facing impact.

Priority when multiple intents appear in one subject:
1. Bug Fixes
2. Feature Development
3. Optimizations

## Language Rules

- Default output language should follow the user's current query language.
- If the user explicitly asks for another language, follow that request.
- Keep section titles and bullet language consistent in one report.

## Writing Rules

- Use commit subjects as source material, but do not copy them verbatim.
- Describe value from the user/business perspective.
- Keep each bullet concise (preferably within 30 Chinese characters, or one short English sentence).
- Start with an action verb, for example:
  - "Completed"
  - "Added"
  - "Enabled"
  - "Optimized"
  - "Fixed"
  - "Improved"
- Avoid pure implementation terms unless needed for clarity.
- Prefer "outcome + impact" wording.

## Output Template

Use this structure:

```markdown
// Weekly Report YYYY/MM/DD - YYYY/MM/DD

### Feature Development
- ...

### Bug Fixes
- ...

### Optimizations
- ...

### Next Week Plan
- ...
```

If the user prefers Chinese output, keep the same structure but translate section titles and bullet points.

When there are no valid commits, use:

```markdown
// Weekly Report YYYY/MM/DD - YYYY/MM/DD

### Feature Development
- No major feature delivery this week.

### Bug Fixes
- Addressed routine maintenance issues and ensured stability.

### Optimizations
- Improved development workflow and prepared technical foundations.

### Next Week Plan
- Continue key feature delivery and close pending quality tasks.
```

## Safe Append Rules (`week.log`)

- Append only. Never truncate existing content.
- Before appending, check whether a report block with the same date range already exists.
- If the same range exists, use one of these modes:
  - `append`: append a revised version as a new block
  - `update-latest`: replace only the latest block for that same range
  - `skip`: do not write and inform the user
- If mode is not provided, ask the user before writing.

## Privacy Rules

- Remove or generalize sensitive internal details in final wording:
  - internal project codenames
  - customer names
  - internal URLs or ticket IDs
- Keep business value while reducing disclosure risk.

## Quality Check (must pass before write)

- No duplicate bullets across sections.
- Every bullet starts with an action verb.
- Every bullet is concise and readable.
- Wording focuses on user/business impact.
- Sections are not empty unless using the no-valid-commits template.

## Rewrite Examples

| Commit Subject | Weekly Report Bullet |
| --- | --- |
| `fix: form reference keeps loading with default fallback` | Fixed a loading issue in form reference fill, improving completion reliability. |
| `feat: support searching member, department, attachment in table` | Added multi-dimension table search to speed up data retrieval. |
| `refactor: clean up upload state handling` | Optimized upload state flow to improve interaction stability and maintainability. |

## Final Response to User

After writing to `week.log`, always tell the user:

1. The report has been appended to `week.log`.
2. A draft "Next Week Plan" is included.
3. Ask if they want wording adjustments (shorter, more business-focused, or Chinese/English style changes).
