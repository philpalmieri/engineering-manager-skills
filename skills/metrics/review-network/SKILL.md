---
name: review-network
description: |-
  Analyze the team's code review network: who reviews whom, reciprocity ratios,
  network density, and time-series comparison. Surfaces siloed review patterns
  and shows improvement toward distributed collaboration. Use when asked about
  review dynamics, collaboration patterns, or team connectivity.
---

# Review Network Analysis

Analyze the team's review relationships to surface collaboration patterns, detect silos, and track improvement toward distributed code review.

## When to Trigger

- "review network", "who reviews whom", "collaboration graph"
- "review dynamics", "review distribution", "are we siloed?"
- As part of a team health report

## Data Sources

- `data/YYYY-MM/{login}-reviews.json` — reviews given by each person
- `data/YYYY-MM/{login}-prs.json` — PRs authored (to identify inbound reviews)
- `team.json` — team roster for member list

## Process

### Step 1: Load review data for the requested period

Parse the date range (default: last quarter). For each team member, load their `{login}-reviews.json` files for the relevant months. Extract:
- Who authored the PR being reviewed
- Filter to team-internal reviews only (both author and reviewer are in `team.json` members)

### Step 2: Build the review matrix

Construct an N×N matrix where:
- Rows = reviewer
- Columns = PR author
- Values = review count

```
| Reviewer | alice | bob | carol | dave | Total Given |
|----------|-------|-----|-------|------|-------------|
| alice    |   ·   |  12 |   8   |   3  |     23      |
| bob      |   5   |  ·  |  15   |   7  |     27      |
| carol    |   9   |  18 |   ·   |  11  |     38      |
| dave     |   2   |   4 |   6   |   ·  |     12      |
```

### Step 3: Calculate network metrics

**Network density** (0-1 scale):
- Count of non-zero cells / total possible cells (N×(N-1))
- 1.0 = everyone reviews everyone; low values = siloed

**Collaboration score** (0-1 scale):
- Based on how evenly distributed reviews are across the matrix
- Uses normalized entropy: `H / H_max` where H = Shannon entropy of the flattened matrix
- 1.0 = perfectly uniform; 0.0 = all reviews flow through one person

**Gini coefficient** (0-1 scale):
- Inequality of review distribution
- 0.0 = perfectly equal; 1.0 = one person does all reviews
- Track this over time; decreasing Gini = healthier team

**Bottleneck ratio**:
- Max reviews received by one person / average reviews received
- >2.0 = someone is a bottleneck

### Step 4: Reciprocity analysis

For each pair (A, B), calculate:
- A→B reviews vs B→A reviews
- Ratio (>1 means A reviews B more than reverse)
- Flag highly asymmetric pairs (ratio > 3:1)

### Step 5: Time-series comparison

Compare current period to previous period(s). Show:
- Network density trend (is it improving?)
- Gini trend (is distribution getting more equal?)
- New review relationships (pairs that didn't exist before)
- Disappeared relationships (pairs that stopped reviewing each other)

### Step 6: Visualization

Produce an ASCII network diagram showing connection strength:

```
Review Network (Jan-Mar 2026)
Density: 0.86 | Collab: 0.71 | Gini: 0.32

  alice ═══════ bob        (12/5 reviews, reciprocity 2.4:1)
    │  ╲         │
    │    ╲       │
    8      3    15
    │        ╲   │
    ▼          ╲ ▼
  carol ═══════ dave       (6/11 reviews, reciprocity 1:1.8)
```

Use line thickness/characters to indicate volume:
- `═══` heavy (10+ reviews)
- `───` moderate (5-9)
- `···` light (1-4)
- (blank) no relationship

## Output Format

```markdown
# Review Network: {period}

## Network Health
| Metric | Current | Previous | Trend |
|--------|---------|----------|-------|
| Density | 0.86 | 0.71 | 🟢 +21% |
| Collaboration | 0.71 | 0.52 | 🟢 +37% |
| Gini | 0.32 | 0.49 | 🟢 -35% |
| Bottleneck | 1.4 | 2.3 | 🟢 -39% |

## Review Matrix
[matrix table]

## Reciprocity (flagged pairs)
[asymmetric pairs with ratio > 3:1]

## Network Changes
- New connections: [pairs]
- Strengthened: [pairs with >50% increase]
- Weakened: [pairs with >50% decrease]

## Network Diagram
[ASCII visualization]
```

## Interpretation Guide

- **Improving network**: density increasing, Gini decreasing, fewer bottleneck flags
- **Siloed team**: density < 0.5, Gini > 0.5, one person with bottleneck > 3.0
- **Healthy team**: density > 0.7, Gini < 0.35, all pairs have some bidirectional flow
