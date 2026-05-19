# GitHub Skills

Data fetching and analysis skills that pull team activity from GitHub via the `gh` CLI.

## What These Do

These skills form the **data layer** of the system. They fetch, store, and enrich raw GitHub activity (PRs, reviews, issues, sprints, epics) into local JSON files that the metrics and management skills consume.

## Skills

| Skill | What it does |
|-------|-------------|
| **team-activity** | Fetches authored PRs, reviews given, and issues touched per team member per month. The foundational data source. |
| **enrich-prs** | Adds per-PR review detail: how many rounds of feedback, who dismissed/approved, timeline of reviewer actions. Required for review-cycles and review-network metrics. |
| **team-data-sync** | Smart orchestration: detects what's missing or stale, fetches only the delta, and enriches new PRs. The "just make it current" command. |
| **sprint-status** | Queries your GitHub Project board for current sprint health: goal progress, issue completion, blockers. |
| **sprint-management** | Automates sprint lifecycle transitions: start a new sprint, end the current one, roll over incomplete items. |
| **oncall-review** | Scores a completed on-call shift based on issue engagement, handoff quality, and response patterns. |
| **epic-cleanup** | Audits all active epics for ownership gaps, stale work, and closure candidates. Produces an actionable checklist. |
| **epic-overview** | Generates a grouped overview of all active epics organized by strategic objective or project area. |

## Prerequisites

- **`gh` CLI** authenticated with access to your org's repos and project boards
- **`team.json`** configured with at minimum: `org`, `project_number`, and `members[]` with GitHub logins
- **Write access** to the `data/` directory in your workspace

## Data Structure

```
data/
  team.json                      # your config (required)
  .sync-status.json              # freshness tracking (auto-generated)
  YYYY-MM/
    {login}-prs.json             # authored PRs for the month
    {login}-reviews.json         # reviews given
    {login}-issues.json          # issues created/closed
  enrichment/
    {pr_number}.json             # per-PR review timeline detail
```

## Typical Workflow

```
1. "sync my team data"           → team-data-sync runs, fills data/
2. "how was the sprint?"         → sprint-status queries the project board
3. "review Sam's on-call shift"  → oncall-review scores engagement
4. "audit our epics"             → epic-cleanup flags stale/orphaned work
```

## API Rate Limiting

- `team-activity` fetches 3 queries per member per month (PRs, reviews, issues)
- `enrich-prs` fetches 1 query per team-reviewed PR (can be 50-200+ per month)
- `team-data-sync` minimizes calls by checking `.sync-status.json` freshness
- If a query returns exactly 50 or 100 results, the skill splits the date range to avoid pagination caps

## Assumptions

- Your team's work is primarily in one GitHub org (configured in `team.json`)
- Sprint tracking uses GitHub Projects (not Jira, Linear, etc.)
- Data is stored locally in your workspace (not synced to a remote store)
- Members listed in `team.json` are the full set of people to track
