---
name: team-data-sync
description: |-
  Smart data synchronization that gets team activity data up to date. Detects
  what months are missing or stale, fetches only the delta, and enriches new PRs.
  Use when asked to 'sync data', 'get data up to date', 'refresh team data',
  or before running any metrics/analysis skill.
license: MIT
---

# Team Data Sync

Intelligently synchronize team activity data. Figures out what's missing or stale and fetches only what's needed to bring the local data store current.

## When to Trigger

- "sync data", "get data up to date", "refresh", "update team data"
- "make sure data is current", "fetch latest"
- Before any metrics/analysis skill when data might be stale
- Called by `team-health-report` as Phase 2 (data check)

## Data Store Structure

```
data/
  team.json                    # team config (required)
  YYYY-MM/
    {login}-prs.json           # authored PRs for the month
    {login}-reviews.json       # reviews given
    {login}-issues.json        # issues touched
  enrichment/
    {number}.json              # per-PR review detail
    fetch-manifest.json        # tracks what's been fetched
```

## Process

### Step 1: Inventory current data

Scan the `data/` directory and build a status map:

```
Data inventory:
  Members: 7 (from team.json)
  Months with data: 2025-04 through 2026-04 (13 months)
  Current month (2026-05): PARTIAL (last fetched 2026-05-12)
  Enrichment: 184 files
```

For each month, check:
- Do all members have all 3 files? (prs, reviews, issues)
- Is the current month's data fresh? (fetched within last 3 days)
- Are there new team members without historical data?

### Step 2: Determine what needs fetching

**Rules for staleness:**
- Current month: stale if last fetch was > 3 days ago
- Previous month: stale if it's now past the 5th of the next month and data was fetched before month-end (might be missing late-month activity)
- Older months: assumed complete (don't re-fetch unless user explicitly asks)
- New team members: need backfill from their `joined` date

**Output a sync plan:**
```
Sync plan:
  ✅ 2025-04 through 2026-03: complete (all members, all files)
  ⚠️ 2026-04: missing carol-issues.json (re-fetch needed)
  🔄 2026-05: stale (last fetch: May 12, now May 19)
  🆕 New member "carol" (joined 2025-12): backfill 2025-12 through 2026-03

Actions:
  1. Fetch 2026-05 for all members (current month refresh)
  2. Fetch 2026-04 carol issues (gap fill)
  3. Enrich 2026-05 PRs (after fetch)
```

### Step 3: Execute the plan

Run fetches in order:
1. **Current month** first (most useful, most likely stale)
2. **Gap fills** for missing files
3. **Backfills** for new members (oldest to newest)

For each fetch, use the `team-activity` skill logic:
- `gh search prs --author={login} --owner={org} --created={start}..{end}`
- Cap detection (if 50/100 results, split and re-fetch)
- Save to `data/YYYY-MM/{login}-{type}.json`

### Step 4: Enrich new PRs

After fetching, identify PRs that need enrichment:
- New PRs from the current month fetch that have team reviewers
- Any PRs from gap fills that weren't previously enriched

Run the `enrich-prs` skill logic for the delta.

### Step 5: Freshness metadata

Update a sync metadata file at `data/.sync-status.json`:

```json
{
  "last_sync": "2026-05-19T11:50:00Z",
  "months": {
    "2026-05": { "fetched_at": "2026-05-19T11:50:00Z", "complete": false },
    "2026-04": { "fetched_at": "2026-05-01T09:00:00Z", "complete": true },
    "2026-03": { "fetched_at": "2026-04-02T08:00:00Z", "complete": true }
  },
  "enrichment": {
    "last_run": "2026-05-19T11:52:00Z",
    "total_files": 196,
    "coverage_pct": 82
  }
}
```

### Step 6: Summary report

```
✅ Team data sync complete

Fetched:
  2026-05: 7 members × 3 types = 21 files updated
  2026-04: 1 gap filled (carol-issues.json)

Enriched:
  12 new PRs enriched (2026-05)
  Total enrichment: 196 files (82% coverage)

Data is current through: 2026-05-19
Next recommended sync: 2026-05-22 (3-day cadence)
```

## Smart Behaviors

### "Just sync" (no arguments)
- Refresh current month
- Fill any gaps detected
- Enrich new PRs
- Report what was done

### "Sync [month]"
- Fetch that specific month for all members
- Enrich PRs from that month

### "Sync last quarter" / "sync Q3"
- Fetch all months in that quarter
- Useful before running `team-health-report`

### "Full refresh"
- Re-fetch current month + previous month
- Re-enrich all PRs in those months (overwrites existing)
- Use when data seems wrong or incomplete

### "Backfill [person]"
- Fetch all months from their `joined` date to present
- Useful when a new team member is added

## Rules

- **Current month is always re-fetched** (it's inherently incomplete until month-end)
- **Past months are assumed stable** unless a gap is detected
- **Enrichment is additive** (never deletes existing files unless "full refresh")
- **Track sync status** to avoid unnecessary re-fetches
- **Report data coverage %** so the user knows if metrics will be reliable
- **Don't block on enrichment** — basic activity data is useful even without enrichment; offer to enrich as a follow-up step if it will take many API calls

## Notes

- Add `data/.sync-status.json` to the data directory (gitignored with the rest of `data/`)
- Rate limits: fetching 7 members × 3 types = 21 API calls per month. Enrichment is 1 call per PR.
- A typical month for a 7-person team might have 80-150 PRs to enrich
- For the first sync on a new machine, ask which months to backfill (don't assume all history is needed)
