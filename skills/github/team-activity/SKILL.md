---
name: team-activity
description: |-
  Fetch GitHub activity (PRs, reviews, issues) for all team members for a given
  time period. Stores results as JSON files organized by month. Use when asked to
  fetch team data, or as a dependency for analysis and snippet skills.
---

# Team Activity Fetch

Query GitHub for each team member's PRs, reviews, and issues over a date range. Save results as JSON files for analysis, snippets, or reporting.

## When to use

- "Fetch team data for May", "pull activity for last 2 weeks", "update team data"
- As a data-gathering step for `weekly-snippet` or `team-analysis` skills

## Prerequisites

- Team roster (`team.json`) with member logins and org
- GitHub CLI (`gh`) authenticated with org access
- A `data/` directory for storing results (will be created if missing)

## Process

### Step 1: Parse the date range

Interpret the user's request into a start and end date:

| Request | Start | End |
|---------|-------|-----|
| "fetch 2026-05" | 2026-05-01 | 2026-05-31 |
| "fetch last week" | prior Monday | prior Sunday |
| "fetch last 2 weeks" | 14 days ago | today |

### Step 2: Ensure output directory exists

```bash
mkdir -p data/YYYY-MM
```

Use the month of the start date. If the range spans two months, create both directories.

### Step 3: Fetch data for each member

For each member in the team roster:

**Authored PRs:**
```bash
gh search prs --author={login} --owner={org} --created={start}..{end} \
  --json number,title,repository,state,createdAt,closedAt,mergedAt,labels \
  --limit 100
```

**Reviews given:**
```bash
gh search prs --reviewed-by={login} --owner={org} --created={start}..{end} \
  --json number,title,repository,author,createdAt \
  --limit 100
```

**Issues touched:**
```bash
gh search issues --assignee={login} --owner={org} --created={start}..{end} \
  --json number,title,repository,state,createdAt,closedAt,labels,assignees \
  --limit 100
```

### Step 4: Cap detection

If any query returns exactly 50 or 100 results, the API likely truncated the response. Split the date range in half and re-query both halves to get complete data. Merge and deduplicate by issue/PR number.

### Step 5: Save results

Write three files per member per month:

```
data/YYYY-MM/
  {login}-prs.json
  {login}-reviews.json
  {login}-issues.json
```

Each file is a JSON array of the raw results from `gh`.

### Step 6: Report summary

```
Fetched data for 3 members (2026-05-01 to 2026-05-14):
  janedoe: 3 PRs, 5 reviews, 2 issues
  bsmith: 8 PRs, 12 reviews, 4 issues
  agarcia: 6 PRs, 9 reviews, 3 issues
  Total: 17 PRs, 26 reviews, 8 issues
  ⚠️ Cap hit on bsmith reviews (split and re-fetched)
```

## Rules

- **Data is additive.** Fetching the same month twice overwrites the files (latest data wins).
- **Parallelize fetches** when possible. Members are independent of each other.
- **Don't filter at fetch time.** Store raw data; let analysis skills handle filtering.
- **Respect rate limits.** If `gh` returns rate limit errors, pause and retry with backoff.
