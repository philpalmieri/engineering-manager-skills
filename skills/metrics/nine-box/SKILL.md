---
name: nine-box
description: |-
  Generate a 9-box talent assessment grid plotting Performance (current output
  vs level expectations) against Growth (trajectory and scope expansion). Uses
  data-driven metrics rather than subjective placement. Use when asked about
  talent mapping, 9-box, performance vs potential, or team calibration.
license: MIT
---

# 9-Box Talent Assessment

Plot team members on a Performance × Growth grid using data-driven metrics. Performance measures current output vs level-appropriate expectations. Growth measures trajectory, scope expansion, and improvement trends.

## When to Trigger

- "9-box", "nine box", "talent grid", "performance vs potential"
- "team calibration", "talent mapping"
- As part of a quarterly team health report

## Data Sources

- `data/YYYY-MM/{login}-prs.json` — output metrics
- `data/YYYY-MM/{login}-reviews.json` — collaboration metrics
- `data/enrichment/{pr_number}.json` — quality metrics
- `team.json` — level data for expectation scaling
- Previous period data — for growth/trajectory calculation

## Process

### Step 1: Define performance axes

**Performance (X-axis):** Current output relative to level expectations
- Aggregates: PRs/month, reviews/month, repo breadth, merge rate, review quality
- Normalized against level expectations (a Junior exceeding at 15 PRs/mo maps differently than a Staff at 15 PRs/mo)
- Score 1-5 scale:
  - 5: Significantly exceeding (A+/A across most metrics)
  - 4: Exceeding in some areas (mix of A and B)
  - 3: Meeting expectations (mostly B grades)
  - 2: Below in some areas (mix of B and C)
  - 1: Below expectations (C/D across most metrics)

**Growth (Y-axis):** Trajectory and scope expansion
- Quarter-over-quarter trend in output metrics
- New repos/projects entered (scope expansion)
- Improvement in review quality (fewer rounds over time)
- Increasing collaboration breadth
- Score 1-5 scale:
  - 5: Strong upward trajectory across multiple dimensions
  - 4: Clear improvement in 2+ areas
  - 3: Stable (meeting expectations consistently)
  - 2: Flat or slightly declining
  - 1: Declining across multiple dimensions

### Step 2: Map to 9-box grid

```
              │ Low Perf (1-2) │ Med Perf (3)   │ High Perf (4-5) │
──────────────┼────────────────┼────────────────┼─────────────────┤
 High Growth  │ Inconsistent   │ High Potential │ Star            │
 (4-5)        │ Performer      │                │                 │
──────────────┼────────────────┼────────────────┼─────────────────┤
 Med Growth   │ Under-         │ Core           │ Strong          │
 (3)          │ performer      │ Contributor    │ Performer       │
──────────────┼────────────────┼────────────────┼─────────────────┤
 Low Growth   │ Action         │ Maintenance    │ Trusted         │
 (1-2)        │ Required       │ Mode           │ Veteran         │
──────────────┴────────────────┴────────────────┴─────────────────┘
```

### Step 3: Context and caveats

For each placement, note:
- What data drove the placement
- Known context that affects the score (new to team, on-call heavy month, leave, etc.)
- Whether the placement is confidence-high or confidence-low (based on data volume)

### Step 4: Level-adjusted commentary

The same raw numbers mean different things at different levels:
- A Junior shipping 20 PRs/month is exceptional
- A Staff shipping 20 PRs/month is expected
- A Staff with LOW output but HIGH review depth and mentorship investment is still performing (their job is force multiplication, not raw output)

### Step 5: Comparison to prior placement

If a previous 9-box exists, show movement:
- `→` moved right (performance improved)
- `↑` moved up (growth trajectory improved)
- `↙` moved down-left (concerning decline)

## Output Format

```markdown
# 9-Box Assessment: {period}

## Grid

```
              │ Low Performance │ Med Performance │ High Performance │
──────────────┼─────────────────┼─────────────────┼──────────────────┤
 High Growth  │                 │ bob, carol      │ alice            │
──────────────┼─────────────────┼─────────────────┼──────────────────┤
 Med Growth   │ dave            │ eve             │ frank            │
──────────────┼─────────────────┼─────────────────┼──────────────────┤
 Low Growth   │                 │                 │                  │
──────────────┴─────────────────┴─────────────────┴──────────────────┘
```

## Level Expectations Reference

| Level | Dimension | Developing | Meeting | Exceeding |
|-------|-----------|-----------|---------|-----------|
| Junior | PRs/mo | <5 | 5-12 | >12 |
| Senior | PRs/mo | <8 | 8-22 | >22 |
| Staff | PRs/mo | <10 | 10-30 | >30 |

## Individual Scores

| Person | Level | Perf Score | Growth Score | Placement | Movement |
|--------|-------|-----------|-------------|-----------|----------|
| alice  | Senior | 4.5 | 4.0 | Star | ↑ (was Strong Performer) |
| bob    | Junior | 3.0 | 4.5 | High Potential | → (was Inconsistent) |

## Context Notes
- **dave**: Low performance score reflects heavy on-call month (2 weeks). Excluding on-call period, metrics are Med performance.
- **bob**: Growth trajectory strong despite low tenure (4 months). Expected to move right.

## Management Actions
- **Stars**: Stretch assignments, visibility opportunities, promotion readiness check
- **High Potential**: Invest in skill development, increase scope, protect time for growth
- **Core Contributors**: Recognize consistency, check engagement and career interests
- **Underperformers**: 1:1 conversation, identify blockers, create improvement plan
```

## Notes

- Minimum 2 months of data required for growth axis (need trend)
- New team members (< 3 months) should be flagged as "Ramping — insufficient data for placement"
- Never use 9-box output as sole input for performance conversations; it's a signal, not a verdict
- If the team is mostly in "Core Contributor" that's healthy — not everyone needs to be a "Star"
- Growth ≠ promotion readiness; it measures trajectory and expansion patterns
