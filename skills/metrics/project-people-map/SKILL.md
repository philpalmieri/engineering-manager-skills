---
name: project-people-map
description: |-
  Map which people are active in which projects/repos to detect silos, measure
  bus factor, and visualize team engagement distribution. Shows project-level
  activity concentration and surfaces where only one person works.
  Use when asked about silos, bus factor, project coverage, or team distribution.
---

# Project-People Activity Map

Visualize the relationship between projects (repos/initiatives) and people to surface silos, single points of failure, and healthy cross-pollination.

## When to Trigger

- "project map", "who works on what", "team distribution"
- "bus factor", "silos", "single point of failure"
- "project coverage", "initiative engagement"
- As part of a team health report

## Data Sources

- `data/YYYY-MM/{login}-prs.json` — repos where PRs were authored
- `data/YYYY-MM/{login}-reviews.json` — repos where reviews were given
- `data/YYYY-MM/{login}-issues.json` — repos where issues were touched
- `team.json` — team roster, optionally `snippet.key_projects` for grouping

## Process

### Step 1: Build the activity matrix

For each team member over the requested period, aggregate activity by repo:
- PRs authored (weight: 3)
- Reviews given (weight: 1)
- Issues touched (weight: 1)

Produce a weighted "engagement score" per person per repo.

### Step 2: Group by project/initiative (optional)

If `team.json` has `snippet.key_projects` with keyword mappings, group related repos under initiative names. Otherwise, show raw repos.

Example grouping:
```
Initiative: "Auth Overhaul"
  Keywords: ["auth", "oauth", "sso"]
  Matched repos: auth-service, oauth-proxy, sso-gateway
  Combined engagement per person
```

### Step 3: Calculate bus factor

For each project/repo:
- **Bus factor** = number of people with meaningful engagement (score > threshold)
- Threshold = at least 2 authored PRs OR 5 reviews in the period
- **Risk level**:
  - 🔴 Bus factor 1 (single point of failure)
  - 🟡 Bus factor 2 (fragile)
  - 🟢 Bus factor 3+ (healthy)

### Step 4: Detect silos

A silo exists when:
- A person works exclusively in 1-2 repos with no overlap with teammates
- A repo has > 80% of activity from a single person
- Cross-review between repos is minimal (people only review within their "zone")

Score each person's **breadth** (distinct repos) and **isolation** (% of activity in their top repo).

### Step 5: Engagement heatmap

Produce an ASCII heatmap:

```
                     | alice | bob | carol | dave | Bus |
  auth-service       |  ███  |  ·  |  ██   |  ·   | 🟡 2 |
  api-gateway        |  ██   | ███ |  █    |  ██  | 🟢 4 |
  data-pipeline      |  ·    |  ·  |  ·    | ███  | 🔴 1 |
  shared-utils       |  █    |  █  |  ██   |  █   | 🟢 4 |
  deploy-infra       |  ·    | ██  |  ·    |  ·   | 🔴 1 |
```

Block intensity:
- `███` = primary contributor (highest score)
- `██` = active contributor
- `█` = light contributor/reviewer
- `·` = no activity

### Step 6: Time-series comparison

If historical data is available, compare:
- Projects that gained new contributors (silo breaking)
- Projects that lost contributors (risk increasing)
- People who expanded scope (growth signal)
- People who narrowed scope (may be intentional or concerning)

## Output Format

```markdown
# Project-People Map: {period}

## Team Health Summary
| Metric | Value |
|--------|-------|
| Active projects | 12 |
| Bus factor = 1 (🔴) | 3 repos |
| Bus factor = 2 (🟡) | 4 repos |
| Bus factor ≥ 3 (🟢) | 5 repos |
| Avg person breadth | 6.2 repos |

## Engagement Heatmap
[heatmap table]

## 🔴 Single Points of Failure
| Repo | Owner | Activity % | Risk |
|------|-------|-----------|------|
| data-pipeline | dave | 94% | No backup |
| deploy-infra | bob | 87% | No backup |

## Silo Detection
| Person | Top Repo | % in Top | Breadth | Signal |
|--------|----------|----------|---------|--------|
| dave   | data-pipeline | 72% | 2 repos | 🔴 Siloed |
| alice  | auth-service | 45% | 6 repos | 🟢 Distributed |

## Changes vs Last Period
- 🟢 carol started reviewing in data-pipeline (was bus factor 1, now 2)
- 🔴 alice stopped touching deploy-infra (was bus factor 2, now 1)
- 🟢 dave expanded to shared-utils (breadth: 2 → 3)

## Recommendations
- [Actionable suggestions based on findings]
```

## Notes

- Filter out repos with < 2 total activities (noise reduction)
- When grouping by initiative, show both grouped and ungrouped views
- Bus factor calculation excludes "reviewer only" activity (must have authored at least 1 PR or issue)
- Consider tenure: new team members naturally have narrower scope initially
