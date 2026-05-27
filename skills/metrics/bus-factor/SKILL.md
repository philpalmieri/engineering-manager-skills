---
name: bus-factor
description: |-
  Identify single points of failure and coverage gaps across team codebases using
  realistic time windows (3+ months). Distinguishes healthy concentration from
  true risk. Calculates meaningful contributor threshold and provides actionable
  recommendations for building backup coverage.
license: MIT
---

# Bus Factor Analysis

Identify single points of failure and coverage gaps across your team's codebases. Focuses on realistic risk assessment using meaningful time windows (not single-month snapshots).

## When to Trigger

- "bus factor", "bus test", "single point of failure"
- "coverage gaps", "who knows what", "backup coverage"
- "risk assessment", "knowledge silos"
- As part of quarterly team health review

## Critical Methodology Rules

### Time Window (MOST IMPORTANT)

**Minimum analysis period: 3 months (1 quarter)**

Why:
- Single months are skewed by PTO, sprint focus, incident response
- People naturally concentrate on specific projects for 2-4 week sprints
- One month makes healthy concentration look like dangerous silos
- Quarterly view shows realistic backup patterns

**When to use different windows:**
- Standard analysis: 3 months (1 quarter)
- Annual review: 6-12 months for long-term patterns
- Emergency assessment: 1 month ONLY if explicitly requested AND with heavy disclaimers about data quality

**NEVER present single-month data as authoritative without explicit user request.**

### Meaningful Engagement Threshold

A person counts as a "meaningful contributor" if they meet EITHER:
- Authored 2+ PRs in the period, OR
- Gave 5+ reviews in the period

Rationale:
- Pure reviewers provide backup knowledge but can't ship fixes alone
- 1 PR could be a one-off drive-by, 2+ shows familiarity
- 5 reviews = engaged with the codebase enough to understand it

### Weighted Activity Scoring

When calculating "primary contributor" or engagement intensity:
- PRs authored: 3x weight (shipping code = deeper knowledge)
- Reviews given: 1x weight (reviewing = understanding, but not hands-on)
- Issues touched: 1x weight (triage/planning engagement)

## Data Sources

**Required:**
- `data/YYYY-MM/{login}-prs.json` — authored PRs per member per month
- `data/YYYY-MM/{login}-reviews.json` — reviews given per member per month
- `data/team.json` — team roster

**Optional:**
- `data/YYYY-MM/{login}-issues.json` — issue activity (adds context but not critical)

## Process

### Step 1: Validate Time Window

- Default to 3 months if user doesn't specify
- If user says "last month" or "this month", confirm: "Bus factor analysis needs at least a quarter of data for accuracy. Should I analyze the last 3 months instead?"
- NEVER silently analyze 1 month and present it as definitive

### Step 2: Load Activity Data

For each team member across the time period:

```
For each repo:
  PRs_authored = count of PRs in that repo
  Reviews_given = count of reviews in that repo
  
  Engagement_score = (PRs_authored * 3) + (Reviews_given * 1)
  
  Is_meaningful_contributor = (PRs_authored >= 2) OR (Reviews_given >= 5)
```

### Step 3: Calculate Bus Factor Per Repo

```
Bus_factor = count of team members where Is_meaningful_contributor = true

Risk_level:
  🔴 Bus factor 0-1: Single point of failure (or abandoned repo)
  🟡 Bus factor 2: Fragile (one person out = risk)
  🟢 Bus factor 3+: Healthy coverage
```

### Step 4: Identify Concentration vs Single Point of Failure

**Key distinction:**
- **Concentration** = One person does most work BUT others can back them up
- **Single Point of Failure** = One person does work AND nobody else touched it

Calculate for each repo:
- Primary contributor = person with highest engagement score
- Primary's share = their score / total team score for that repo
- Backup count = number of other meaningful contributors

Risk assessment:
- Primary share > 70% AND backup count = 0 → 🔴 True single point of failure
- Primary share > 70% AND backup count >= 1 → 🟡 Heavy concentration but covered
- Primary share 40-70% → 🟢 Good distributed ownership
- Primary share < 40% → 🟢 Very distributed (or no clear owner)

### Step 5: Team Member Breadth Analysis

For each person:
- Repo count = number of repos where they're a meaningful contributor
- Primary repos = repos where they have >50% of activity
- Breadth score = repo count / team average

Signals:
- Breadth < 0.5 team average AND primary share > 80% in top repo → 🔴 Siloed specialist
- Breadth < team average → 🟡 Focused contributor
- Breadth >= team average → 🟢 Cross-functional
- Breadth > 1.5x team average → 🟢 Polyglot/reviewer

## Output Format

```markdown
# Bus Factor Analysis: {period}

**Analysis Period:** {start_date} to {end_date} ({N} months)  
**Team Size:** {N} members  
**Active Repos:** {N} repositories with activity

---

## Executive Summary

| Risk Level | Count | Repos |
|------------|-------|-------|
| 🔴 Single Point of Failure (0-1) | {N} | {list} |
| 🟡 Fragile Coverage (2) | {N} | {list} |
| 🟢 Healthy Coverage (3+) | {N} | {list} |

**Overall Health:** {Green/Yellow/Red} — {brief assessment}

---

## Coverage by Repository

| Repository | Bus Factor | Primary | Primary % | Backups | Risk |
|------------|-----------|---------|-----------|---------|------|
| auth-service | 4 | alice | 45% | bob, carol, dave | 🟢 Healthy |
| data-pipeline | 1 | dave | 94% | — | 🔴 No backup |
| api-gateway | 3 | bob | 52% | alice, carol | 🟢 Covered |

---

## 🔴 Critical Risks (if any)

### {repo-name}
- **Bus factor:** 1
- **Primary:** @username (94% of activity)
- **Backup coverage:** None
- **Activity:** 12 PRs, 3 reviews (all from primary)
- **Risk:** If @username is unavailable, no one else can maintain this service
- **Recommendation:** {specific action}

---

## 🟡 Areas of Concentration (if any)

### {repo-name}
- **Bus factor:** 2-3
- **Primary:** @username (68% of activity)
- **Backups:** @user2 (4 PRs), @user3 (8 reviews)
- **Assessment:** Heavy concentration but adequate backup exists
- **Recommendation:** {if needed, or "Monitor" if acceptable}

---

## Team Breadth Analysis

| Member | Repos Touched | Primary Repos | Breadth | Signal |
|--------|---------------|---------------|---------|--------|
| alice | 6 | 1 (auth) | 1.2x | 🟢 Distributed |
| bob | 5 | 2 (api, infra) | 1.0x | 🟢 Balanced |
| dave | 2 | 1 (pipeline) | 0.4x | 🔴 Siloed |

**Team Average Breadth:** {N} repos per person

---

## Recommendations

### Immediate Actions (if any 🔴 risks exist)
1. {Specific action for specific repo/person}
2. {Cross-training recommendation}

### Medium-Term Health (for 🟡 areas)
1. {Spread review load}
2. {Pair programming opportunities}

### Positive Patterns to Maintain
- {Callout good coverage examples}
- {Recognize cross-functional contributors}

---

## Methodology Notes

- **Analysis period:** {N} months (quarterly view for realistic patterns)
- **Meaningful contributor threshold:** 2+ PRs authored OR 5+ reviews given
- **Engagement weighting:** PRs (3x), Reviews (1x)
- **Risk levels:** Bus factor 0-1 (🔴), 2 (🟡), 3+ (🟢)
```

## Comparison Formats (when analyzing trends)

If running analysis on multiple periods for comparison:

```markdown
## Trend Analysis: {period1} vs {period2}

### Risk Changes

| Repo | Previous | Current | Change |
|------|----------|---------|--------|
| data-pipeline | 🔴 1 | 🟡 2 | ✅ carol added backup |
| deploy-infra | 🟡 2 | 🔴 1 | ⚠️ alice stopped contributing |

### Team Breadth Evolution

| Member | Previous | Current | Change |
|--------|----------|---------|--------|
| dave | 2 repos | 4 repos | ✅ Expanded scope |
| alice | 7 repos | 4 repos | ℹ️ Focused (may be intentional) |
```

## Important Caveats to Include

Always include these disclaimers in your analysis:

1. **"Bus factor measures contribution activity, not knowledge."** Someone who reviewed 5 PRs may understand the code but can't ship fixes without repo access or ramp-up time.

2. **"Concentration isn't always bad."** Having a clear owner who knows a system deeply is valuable IF they have backup.

3. **"Low activity ≠ abandoned."** Stable, mature services naturally have less activity. Distinguish between "no one touched it because it's stable" vs "no one CAN touch it except Dave."

4. **"New team members skew the data."** Someone who joined 2 months ago will naturally have narrow scope. Note tenure when flagging "siloed" individuals.

5. **"PTO/leave affects windows."** If someone took 3 weeks off during the analysis period, their metrics will look lower. Adjust interpretation accordingly.

## Example Triggers & Responses

**User:** "Run a bus factor analysis"
→ Analyze last 3 months, full format

**User:** "Bus factor for May"
→ Respond: "Bus factor needs at least a quarter of data for accuracy. Should I analyze Mar-May (3 months) instead?"

**User:** "Who can work on {repo-name}?"
→ Show contributors for that specific repo with activity breakdown

**User:** "Is dave siloed?"
→ Show dave's breadth analysis across repos with context

**User:** "Quick bus check"
→ Show just the executive summary table with risk counts

## Integration with Other Skills

- Run as part of **team-health-report** for quarterly review
- Use data fetched by **team-data-sync** (no separate fetch needed)
- Complements **review-network** (who reviews whom) and **nine-box** (performance assessment)
- Informs **review-cards** by highlighting areas where people should expand

## Key Takeaway

**The goal is realistic risk assessment, not shaming people for focus areas.**

Frame findings as:
- "Where do we need to build backup coverage?"
- "Who should we pair up for knowledge transfer?"
- "Which stable systems are actually fine with one expert?"

Not:
- "Dave is a silo" (accusatory)
- "This repo is abandoned" (without checking if it's just stable)
- "Everyone should touch everything" (unrealistic)
