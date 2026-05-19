---
name: review-cards
description: |-
  Generate quarterly review cards grading each engineer across multiple dimensions:
  output, velocity, collaboration, quality, breadth, review depth, responsiveness,
  and mentorship. Grades are level-adjusted (expectations scale with seniority).
  Use when asked for review cards, quarterly grades, or individual performance cards.
---

# Engineer Review Cards

Generate grade-based review cards for each team member, assessing performance across multiple dimensions with level-adjusted expectations.

## When to Trigger

- "review cards", "quarterly grades", "performance cards"
- "how is [person] doing?", "grade the team"
- "level-adjusted performance", "expectations by level"
- As part of a team health report

## Data Sources

- `data/YYYY-MM/{login}-prs.json` — output and velocity
- `data/YYYY-MM/{login}-reviews.json` — collaboration and breadth
- `data/YYYY-MM/{login}-issues.json` — issue engagement
- `data/enrichment/{pr_number}.json` — quality (review rounds), review depth
- `team.json` — level information for expectation scaling

## Process

### Step 1: Define level expectations

Expectations scale with level. Define thresholds per level:

| Dimension | Junior (L1-L2) | Mid (L3-L4) | Senior (L5-L6) | Staff+ (L7+) |
|-----------|---------------|-------------|----------------|--------------|
| PRs/month | 5-12 | 8-22 | 10-30 | 10-30+ |
| Reviews/month | 5-20 | 15-50 | 25-70 | 25-70+ |
| Repo breadth | 1-4 | 3-8 | 5-12 | 5-12+ |

These are configurable. If `team.json` has level data, map members to the appropriate tier. If levels are non-standard (e.g., "Senior", "IC3", "L9"), ask the user to confirm the mapping or infer from common patterns.

### Step 2: Calculate raw metrics per person per quarter

| Metric | Calculation |
|--------|-------------|
| **Output** | PRs authored per month (averaged over quarter) |
| **Velocity** | Median time-to-merge (hours) |
| **Collaboration** | Reviews given to teammates per month |
| **Quality** | Average review rounds before merge (lower = better) |
| **Breadth** | Distinct repos with activity |
| **Review Depth** | % of reviews with written comments or change requests (vs rubber stamps) |
| **Responsiveness** | Median hours to first review on teammates' PRs |
| **Mentorship** | Review count weighted 2× for reviews given to newer team members (< 12 months tenure) |

### Step 3: Assign grades

Map each metric to a grade based on level-adjusted expectations:

| Grade | Meaning |
|-------|---------|
| A+ | Significantly exceeding level expectations |
| A | Exceeding expectations |
| B+ | Meeting expectations with strength |
| B | Meeting expectations |
| C | Slightly below expectations |
| D | Below expectations |

Grading uses the level-appropriate thresholds:
- A+: > 130% of "exceeding" threshold
- A: In "exceeding" range
- B+: High end of "meeting" range
- B: In "meeting" range
- C: In "developing" range
- D: Below "developing" range

### Step 4: Calculate overall grade

Weighted average of dimensions:
- Output: 20%
- Velocity: 10%
- Collaboration: 20%
- Quality: 15%
- Breadth: 10%
- Review Depth: 10%
- Responsiveness: 10%
- Mentorship: 5%

### Step 5: Growth trajectory

Compare current quarter to previous 1-2 quarters:
- 📈 Growing: overall grade improved
- ➡️ Stable: same grade (±0.3)
- 📉 Declining: overall grade dropped

### Step 6: Multi-quarter view

Show a table with grades across the last 3-4 quarters to visualize trends at a glance.

## Output Format

```markdown
# Engineer Review Cards: {quarter}

## Team Grading Summary
| Person | Level | Output | Velocity | Collab | Quality | Breadth | Depth | Response | Mentor | Overall | Trend |
|--------|-------|--------|----------|--------|---------|---------|-------|----------|--------|---------|-------|
| alice  | Sr    | A      | A+       | B      | A       | A       | B+    | A        | C      | **A-**  | 📈    |
| bob    | Mid   | B      | C        | A      | B       | B       | A     | B        | A      | **B+**  | ➡️    |

---

## Individual Cards

### alice (Senior Engineer)

| Quarter | Output | Velocity | Collab | Quality | Breadth | Depth | Response | Mentor | Overall |
|---------|--------|----------|--------|---------|---------|-------|----------|--------|---------|
| Q1      | B      | A        | C      | B       | B       | —     | —        | —      | B       |
| Q2      | A      | A        | B      | A       | A       | B     | A        | C      | A-      |
| Q3      | A      | A+       | B      | A       | A       | B+    | A        | C      | A-      |

**Growth trajectory:** 📈 Strong upward trend

**Strengths:** Velocity, output, breadth
**Growth areas:** Mentorship (low review investment in newer members)

**Domain expertise:**
- 🔵 Owner: auth-service, api-gateway
- 🟢 Contributor: shared-utils, deploy-infra
```

## Notes

- Quarters with < 1 month of data get "—" (insufficient)
- New team members (< 3 months) get a ramp-adjusted grade (expectations halved)
- The quality metric inverts: lower rounds = better grade
- The velocity metric inverts: lower TTM = better grade
- "—" for metrics where enrichment data isn't available
- If levels aren't in team.json, ask the user before proceeding
