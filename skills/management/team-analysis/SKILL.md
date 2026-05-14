---
name: team-analysis
description: |-
  Generate a team activity report for a date range. Analyzes PR volume, review patterns,
  project distribution, and individual contributions. Use for monthly reports, quarterly
  reviews, or ad-hoc team health checks.
---

# Team Analysis

Generate an activity report analyzing team contributions over a date range. Combines fetched GitHub data with vault context to produce a narrative report.

## When to use

- "Analyze team last month", "team report for Q3", "how did the team do in April?"
- "Analyze {member} last 2 weeks" (individual focus)

## Prerequisites

- Fetched team activity data in `data/YYYY-MM/` (use `team-activity` skill if missing)
- Team roster (`team.json`)

## Process

### Step 1: Resolve the date range

Parse the request into concrete dates. Support:

| Request | Resolution |
|---------|------------|
| "analyze team 2026-05" | May 1-31 |
| "analyze team last 2 weeks" | 14 days back from today |
| "analyze {member}" | Check `reports/index.json` for last report covering this member; analyze from that date to now |
| "analyze {member} last week" | Last 7 days |

### Step 2: Load data

Read the relevant JSON files from `data/YYYY-MM/` directories. If data is missing for the requested range, fetch it first using the `team-activity` skill.

### Step 3: Analyze

**Team-level metrics:**
- Total PRs authored, merged, and reviewed
- PR distribution by repo (where is work happening?)
- Review coverage (who's reviewing whose work?)
- Issue throughput (opened vs. closed)

**Individual-level metrics:**
- PRs authored and merged (count and titles)
- Reviews given (count and to whom)
- Issues closed
- Notable patterns: breadth across repos, review depth, velocity trends

**Qualitative observations:**
- Project momentum: which projects are moving fast vs. stalled?
- Collaboration patterns: is everyone reviewing, or is it concentrated?
- Risk signals: anyone with zero reviews? Anyone with lots of open but few merged?

### Step 4: Generate report

Write a markdown report:

```markdown
# Team Activity Report: {date_range}

## Summary
- {N} PRs authored ({M} merged) across {R} repositories
- {X} reviews given, {Y} issues closed
- Top contributors: ...

## By Project
### {Project/Repo Name}
- Key PRs and what they accomplished
- Who worked on it

## By Member
### {Member Name}
- PRs: [list with links]
- Reviews: [count, notable ones]
- Observations: [patterns, recognition, concerns]

## Observations
- [Cross-cutting patterns, collaboration health, risk signals]
```

### Step 5: Save and index

1. Save report to `reports/` directory with descriptive filename:
   - Team: `analysis-team-YYYY-MM.md`
   - Individual: `analysis-{login}-YYYY-MM.md`

2. Update `reports/index.json`:
```json
{
  "reports": [
    {
      "id": "rpt-{timestamp}",
      "type": "team|individual",
      "member": "{login or null}",
      "from": "YYYY-MM-DD",
      "to": "YYYY-MM-DD",
      "file": "analysis-....md",
      "generated_at": "ISO-8601"
    }
  ]
}
```

This index lets future "analyze {member}" calls pick up where the last report left off.

## Rules

- **Data first.** If data isn't fetched for the requested range, fetch it before analyzing. Don't guess.
- **No ranking or stack-ranking.** Don't compare members against each other. Analyze each person's contributions on their own merits.
- **Context matters.** Someone with 2 PRs who shipped a complex migration is more notable than someone with 15 trivial dependency bumps. Weight quality over quantity.
- **Flag what you can't see.** GitHub data misses meetings, design work, mentoring, code review depth, and cross-team coordination. Note these gaps.
- **Fiscal year awareness.** If the team uses a non-standard fiscal year (e.g., Q1 = Jul-Sep), respect that mapping from the team roster config.
