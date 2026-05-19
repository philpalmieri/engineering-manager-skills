---
name: team-health-report
description: |-
  Orchestration skill that runs all metrics analyses and produces a unified team
  health dashboard. Combines review network, review cycles, project-people map,
  contribution velocity, review cards, and 9-box into a single comprehensive
  report with period-over-period comparison. Use when asked to 'run team health',
  'full metrics report', 'quarterly team assessment', or 'team dashboard'.
---

# Team Health Report (Orchestration)

Run all metrics skills and produce a unified team health dashboard with period-over-period comparisons.

## When to Trigger

- "team health report", "run team health", "full metrics"
- "quarterly assessment", "team dashboard", "metrics report"
- "how's the team doing?" (when comprehensive answer desired)

## Data Sources

All data sources from constituent skills:
- `data/YYYY-MM/{login}-prs.json`
- `data/YYYY-MM/{login}-reviews.json`
- `data/YYYY-MM/{login}-issues.json`
- `data/enrichment/{pr_number}.json`
- `team.json`

## Process

### Phase 1: Determine scope

Ask the user or infer:
- **Period**: Quarter (default), month, or custom date range
- **Comparison**: Auto-select previous same-length period
- **Depth**: Full (all sections) or summary (dashboard + flags only)

### Phase 2: Data check

Verify data exists for the requested period:
- Check `data/YYYY-MM/` directories cover the full range
- Check enrichment data availability (what % of PRs are enriched?)
- If data is missing, offer to fetch it first via `team-activity` skill

Report data coverage:
```
Data coverage: 2026-01 through 2026-03
Enrichment: 142/184 PRs enriched (77%)
All 7 team members have activity data.
```

### Phase 3: Run analyses (in order)

Execute each metrics skill's logic:

1. **Review Network** → network density, Gini, bottleneck ratio, matrix
2. **Review Cycles** → avg rounds per person, trends, outlier PRs
3. **Project-People Map** → bus factor, silo detection, breadth scores
4. **Contribution Velocity** → TTM, throughput, responsiveness
5. **Review Cards** → per-person quarterly grades
6. **9-Box** → talent grid placement

### Phase 4: Synthesize dashboard

Combine all outputs into a unified report with:

**Executive summary** (3-5 bullet points):
- Headline team health signal (improving/stable/declining)
- Biggest win this period
- Biggest risk or concern
- Notable individual movements

**Key metrics table** (the "vital signs"):

| Metric | Current | Previous | Trend | Target |
|--------|---------|----------|-------|--------|
| Network density | 0.86 | 0.71 | 🟢 +21% | >0.7 |
| Gini coefficient | 0.32 | 0.49 | 🟢 -35% | <0.4 |
| Team TTM p50 | 18h | 24h | 🟢 -25% | <24h |
| PRs/FTE/month | 14.2 | 12.8 | 🟢 +11% | >10 |
| Avg review rounds | 1.3 | 1.6 | 🟢 -19% | <2.0 |
| Bus factor 1 repos | 3 | 5 | 🟢 -40% | 0 |
| First review p50 | 4h | 6h | 🟢 -33% | <8h |

**Individual highlight cards** (per person, 2-3 lines):
- Current grade, trajectory, standout metric, concern if any

### Phase 5: Generate insights

Go beyond the numbers. Identify:

**Patterns:**
- Is the team trending healthier or declining?
- Are specific pairs over-indexing on each other?
- Is scope expanding or contracting?

**Coaching signals:**
- Who needs attention? (declining metrics, high review cycles)
- Who deserves recognition? (growth trajectory, network contributions)
- Who is ready for more scope? (exceeding consistently, narrow breadth)

**Structural observations:**
- Any single points of failure that need addressing?
- Is review load balanced or bottlenecked?
- Are new team members ramping appropriately?

### Phase 6: Recommendations

Actionable next steps (3-5 max):
- Specific, addressed to the manager
- Tied to data (not generic advice)
- Prioritized by impact

Example:
```
1. Pair dave with carol on data-pipeline reviews (bus factor 1 → 2, carol has capacity)
2. Recognize alice in team standup (A+ velocity for 3 consecutive months)
3. Check in with bob about TTM regression (45h → 72h this month, was 22h prior)
```

## Output Format

The full report follows this structure:

```markdown
# Team Health Report: {period}
> Compared against: {previous_period}

## Executive Summary
- [3-5 bullet synthesis]

## Vital Signs
[key metrics table]

## Review Network
[condensed network output — matrix, density, key flags]

## Contribution Velocity
[per-person velocity table, team trends]

## Review Quality
[review cycles summary, outliers]

## Project Coverage
[heatmap, bus factor flags, silo alerts]

## Individual Cards
[grade table for all members]

## 9-Box Grid
[current placement, movement from prior]

## Insights & Recommendations
[synthesized coaching signals and action items]
```

## Output Location

Write the report to:
- `reports/metrics/team-health-{YYYY-MM-DD}.md` (in the data workspace)
- Optionally: the user's vault at a location they specify

## Cadence

Recommended: run quarterly (aligned to fiscal year quarters). Can run monthly for tighter feedback loops. Monthly reports can be lighter (skip 9-box and review cards, focus on velocity and network).

## Notes

- This is a READ-ONLY analysis skill. It produces reports but never modifies source data.
- If enrichment data is sparse (< 50% of PRs enriched), flag it and note which metrics are affected.
- The report should be useful standalone; a reader shouldn't need to open constituent reports.
- Keep the executive summary short enough to scan in 30 seconds.
