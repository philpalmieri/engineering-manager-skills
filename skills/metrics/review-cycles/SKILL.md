---
name: review-cycles
description: |-
  Track PR review rounds (feedback cycles) before merge per person and over time.
  Surfaces who consistently needs many iterations vs ships cleanly, and whether
  individuals are improving. Uses enrichment data for accurate round counts.
  Use when asked about review quality, iteration patterns, or PR friction.
license: MIT
---

# Review Cycles Analysis

Track the number of review rounds (feedback → update → re-review cycles) each person goes through before their PRs merge. Identify patterns, outliers, and improvement trends.

## When to Trigger

- "review cycles", "review rounds", "how many iterations"
- "PR friction", "review quality", "who takes the most feedback"
- "is [person] improving?", "code quality trends"
- As part of a team health report

## Data Sources

- `data/enrichment/{pr_number}.json` — per-PR review round data including:
  - `review_rounds`: total round count
  - `changes_requested`: CR events
  - `dismissals`: review dismissals (often indicates a revision)
  - `reviewer_actions`: who did what
- `data/YYYY-MM/{login}-prs.json` — PR metadata (dates, merge status, number)
- `team.json` — team roster

## Process

### Step 1: Load PR data for the period

For each team member, load their authored PRs for the date range. For each merged PR, look up the enrichment data by PR number at `data/enrichment/{number}.json`.

### Step 2: Calculate per-person metrics

For each team member, compute:

| Metric | Description |
|--------|-------------|
| Avg rounds | Mean review rounds across all their merged PRs |
| Median rounds | P50 rounds (more robust to outliers) |
| Max rounds | Worst case (highest friction PR) |
| % single-round | PRs that merged after first review (no iterations) |
| Trend | Month-over-month change in avg rounds |

### Step 3: Team benchmarks

Calculate team-wide baselines:
- Team average rounds
- Team median rounds
- Standard deviation (to identify outliers)

Flag individuals > 1 std dev above team mean as "high iteration" and those consistently below as "clean shippers."

### Step 4: Time-series by person

Build a month-by-month table showing each person's average rounds:

```
| Month    | alice | bob  | carol | dave | Team Avg |
|----------|-------|------|-------|------|----------|
| 2026-01  | 1.2   | 2.8  | 1.5   | 1.0  | 1.6      |
| 2026-02  | 1.1   | 2.3  | 1.4   | 1.2  | 1.5      |
| 2026-03  | 1.0   | 1.8  | 1.3   | 1.1  | 1.3      |
```

Note: Only include months with 3+ enriched PRs for statistical validity. Show sample size in parentheses.

### Step 5: Identify patterns

**Improvement signals:**
- Decreasing avg rounds over 3+ months
- Increasing % single-round merges

**Concern signals:**
- Consistently > 2.5 avg rounds (team should average ~1.5)
- High dismissal count (may indicate premature review requests)
- One reviewer responsible for most iterations (pairing issue vs code quality issue)

**Coaching opportunities:**
- Person with high rounds + specific reviewer always requesting changes → targeted pairing/mentoring
- Person with high rounds across all reviewers → broader code quality discussion

### Step 6: Reviewer contribution to cycles

Cross-reference with reviewer actions:
- Which reviewers drive the most iteration? (High dismissal/CR rate)
- Is one reviewer a "rubber stamp" (always approves first round)?
- Are iterations productive (PRs improve) or churn (same feedback repeated)?

## Output Format

```markdown
# Review Cycles Report: {period}

## Team Overview
| Metric | Value |
|--------|-------|
| Team avg rounds | 1.5 |
| Team median | 1.0 |
| % single-round merges | 62% |
| Trend | 🟢 Improving (-15% vs prior period) |

## Per-Person Breakdown

| Person | Avg Rounds | Median | % Clean | Trend | Signal |
|--------|-----------|--------|---------|-------|--------|
| alice  | 1.1 (12 PRs) | 1.0 | 75% | ➡️ Stable | Clean shipper |
| bob    | 2.3 (8 PRs) | 2.0 | 25% | 🟢 -22% | Improving |
| carol  | 1.4 (15 PRs) | 1.0 | 60% | ➡️ Stable | Normal |
| dave   | 1.0 (5 PRs) | 1.0 | 80% | ➡️ Stable | Clean shipper |

## Month-over-Month Trend
[table with monthly averages]

## Outlier PRs (5+ rounds)
- PR #1234 (bob): 8 rounds, reviewer: carol — [title]
- PR #5678 (carol): 5 rounds, reviewer: alice — [title]

## Coaching Signals
- **bob**: High iteration count driven primarily by carol's reviews (6 of 8 PRs). Consider pairing session on [common feedback theme].
- **dave**: Consistently clean but low volume — may be over-polishing before requesting review.
```

## Notes

- PRs with 0 enrichment data (not yet cached) are excluded from analysis
- Only count merged PRs (open/closed-without-merge are excluded)
- "Round" = a review event followed by new commits followed by another review event
- Minimum 3 enriched PRs per person per month for trend validity
