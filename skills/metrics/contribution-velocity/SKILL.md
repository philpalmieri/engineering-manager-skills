---
name: contribution-velocity
description: |-
  Track PR throughput, time-to-merge, and review responsiveness per person and
  team-wide. Shows delivery speed trends and identifies bottlenecks in the
  review-to-merge pipeline. Use when asked about velocity, merge times,
  throughput, or delivery speed.
license: MIT
---

# Contribution Velocity

Measure how fast work flows through the team: PR throughput, time-to-merge, and review responsiveness. Identifies where the pipeline slows down.

## When to Trigger

- "velocity", "time to merge", "how fast are we shipping"
- "throughput", "delivery speed", "merge times"
- "review responsiveness", "how long do PRs sit"
- As part of a team health report

## Data Sources

- `data/YYYY-MM/{login}-prs.json` — PR metadata including created/merged dates
- `data/YYYY-MM/{login}-reviews.json` — review timestamps
- `data/enrichment/{pr_number}.json` — detailed review timeline
- `team.json` — team roster

## Process

### Step 1: Calculate time-to-merge (TTM)

For each merged PR in the period:
- TTM = merged_at - created_at (in hours)
- Compute p50 (median), p75, p90 per person
- Compute team-wide percentiles

### Step 2: PR throughput

Per person per month:
- PRs authored
- PRs merged
- Merge rate (merged / authored)
- Net output (merged PRs, not just opened)

### Step 3: Review responsiveness

For reviews given by each team member:
- Time from PR creation to first team review (hours)
- This measures how long PRs sit before getting attention
- Compute per-reviewer and team-wide

### Step 4: Pipeline breakdown

For enriched PRs, break TTM into stages:
- **Wait for first review**: created → first review event
- **Iteration time**: first review → final approval
- **Merge lag**: final approval → actual merge

This shows WHERE time is spent:
- Long wait = not enough reviewers / low priority
- Long iteration = code quality issues or over-engineering reviews
- Long merge lag = CI/deployment issues or author delay

### Step 5: Trend analysis

Month-over-month tables showing:
- TTM p50 per person (are they getting faster?)
- Throughput per person (are they producing more?)
- Review response time (is the team more responsive?)

Flag:
- 🟢 Improving: >20% faster than 3-month average
- 🔴 Degrading: >20% slower than 3-month average
- ⚠️ Spike: single month outlier (p90 > 3× p50)

### Step 6: Outlier detection

Flag PRs that took > 1 week to merge:
- Is there a pattern? (specific repos, specific authors, specific reviewers)
- Are long-lived PRs getting longer or shorter over time?

## Output Format

```markdown
# Contribution Velocity: {period}

## Team Summary
| Metric | Current | Previous | Trend |
|--------|---------|----------|-------|
| TTM p50 | 18h | 24h | 🟢 -25% |
| TTM p90 | 96h | 120h | 🟢 -20% |
| PRs merged/FTE/month | 14.2 | 12.8 | 🟢 +11% |
| First review time (p50) | 4h | 6h | 🟢 -33% |

## Per-Person Velocity

| Person | TTM p50 | TTM p90 | PRs/mo | Merge % | Response | Trend |
|--------|---------|---------|--------|---------|----------|-------|
| alice  | 8h | 24h | 18 | 94% | 3h | 🟢 |
| bob    | 22h | 72h | 12 | 88% | 8h | ➡️ |
| carol  | 15h | 48h | 22 | 91% | 2h | 🟢 |
| dave   | 45h | 168h | 8 | 75% | 12h | 🔴 |

## Monthly Trends (TTM p50, hours)

| Month | alice | bob | carol | dave | Team |
|-------|-------|-----|-------|------|------|
| Jan   | 12    | 28  | 18    | 52   | 22   |
| Feb   | 10    | 24  | 16    | 48   | 20   |
| Mar   | 8     | 22  | 15    | 45   | 18   |

## Pipeline Breakdown (team averages)
| Stage | Hours | % of TTM |
|-------|-------|----------|
| Wait for first review | 4h | 22% |
| Iteration | 8h | 44% |
| Merge lag | 6h | 33% |

## Long-tail PRs (> 1 week)
- PR #1234 (dave): 12 days — blocked on CI, repo: data-pipeline
- PR #5678 (bob): 8 days — waiting on external dependency

## Signals
- dave's TTM is 2.5× team average — check for blockers or large PR pattern
- alice shipping fastest with highest volume — strong signal
- Team first-review time improving: 6h → 4h over 3 months
```

## Notes

- TTM uses only merged PRs (exclude open/closed-without-merge)
- Exclude bot PRs and automated dependency updates
- Weekend/holiday hours are included (calendar time, not business time)
- For sparse months (< 3 merged PRs for a person), mark as "insufficient data"
- Consider PR size: a 500-line refactor should take longer than a 10-line fix
