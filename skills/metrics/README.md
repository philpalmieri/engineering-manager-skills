# Metrics Skills

Data-driven team health analysis: collaboration patterns, delivery velocity, talent assessment, and trend visualization.

## What These Do

These skills transform raw GitHub activity data into actionable insights about team dynamics. They help you answer questions like "are we siloed?", "is review quality improving?", "who needs a stretch opportunity?", and "where's our bus factor risk?"

## Skills

| Skill | What it does |
|-------|-------------|
| **review-network** | Builds a collaboration graph (who reviews whom). Measures network density, reciprocity, Gini coefficient, and detects silos. Shows 1:1 → 3:3 progression. |
| **review-cycles** | Tracks PR feedback rounds before merge per person and trending over time. Surfaces people with high round counts and shows improvement trajectories. |
| **project-people-map** | Maps projects to people working on them. Calculates bus factor, detects single-person silos, and shows knowledge distribution as a heatmap. |
| **contribution-velocity** | Measures time-to-merge (P50/P75/P95), weekly throughput, and review responsiveness. Tracks trends period-over-period. |
| **review-cards** | Quarterly grade cards per person: delivery, review quality, collaboration, growth. Level-adjusted expectations (IC2 expectations differ from IC4). |
| **nine-box** | 9-box talent grid placement: performance (x-axis) vs growth trajectory (y-axis). Uses quantitative signals, not gut feel. |
| **team-health-report** | Orchestration skill that runs ALL metrics and produces a unified dashboard with period-over-period comparison. |

## Prerequisites

- **Team activity data** in `data/YYYY-MM/` (fetched via `team-activity` skill)
- **Enrichment data** in `data/enrichment/` (fetched via `enrich-prs` skill)
- **`team.json`** with members, levels, and start dates
- At least **1 month of data** for basic metrics; **3+ months** for meaningful trends

### Quick Data Setup

If you don't have data yet:
```
> sync my team data              # fetches current + previous month
> sync last quarter              # fetches 3 months for trend analysis
```

## Data Dependencies

```
review-network       ← enrichment/*.json + {login}-reviews.json
review-cycles        ← enrichment/*.json (review_rounds field)
project-people-map   ← {login}-prs.json (repo field)
contribution-velocity← {login}-prs.json (created/merged dates) + enrichment
review-cards         ← ALL data sources (composite score)
nine-box             ← ALL data sources + team.json (levels, start dates)
team-health-report   ← ALL of the above (orchestration)
```

## Output

Reports are written to `reports/metrics/` as markdown files:

```
reports/metrics/
  team-dashboard-YYYY-MM.md       # unified health report
  review-dynamics-YYYY-MM.md      # network + cycles combined
  review-cards-YYYY-QN.md         # quarterly grade cards
  9box-assessment-YYYY-QN.md      # talent grid
  knowledge-map-YYYY-MM.md        # project-people analysis
  month-over-month-YYYY-MM.md     # velocity trends
```

## Typical Workflow

```
Quarterly review:
1. "sync last quarter"                    → ensures data is complete
2. "run team health report for Q1"        → full dashboard
3. "generate review cards for Q1"         → individual assessments
4. "do 9-box for the team"               → talent grid

Monthly check-in:
5. "show review network for last month"   → collaboration health
6. "how are review cycles trending?"      → feedback efficiency

Ad-hoc:
7. "bus factor on our projects?"          → project-people-map
8. "Sam's velocity this quarter?"         → contribution-velocity (filtered)
```

## Key Metrics Explained

| Metric | What it measures | Healthy range |
|--------|-----------------|---------------|
| Network density | % of possible reviewer pairs that are active | > 0.4 |
| Gini coefficient | Evenness of review distribution (0=equal, 1=one person does all) | < 0.35 |
| Bottleneck ratio | % of reviews going through one person | < 30% |
| Review rounds | Feedback cycles before merge | 1-2 typical; 4+ is a signal |
| Bus factor | Min people who'd need to leave to stall a project | > 2 |
| P50 TTM | Median time from PR open to merge | < 24h for team PRs |

## Assumptions

- Team-internal reviews are more meaningful than external reviews for collaboration metrics
- Level expectations differ (an IC2 with 3 review rounds is learning; an IC4 with 3 rounds needs investigation)
- Data is fetched via `team-activity` and `enrich-prs` skills and stored as JSON
- Metrics improve gradually; single-month snapshots are less useful than 3+ month trends
- The 9-box and review-cards skills involve qualitative judgment; treat outputs as starting points for conversation, not final assessments
