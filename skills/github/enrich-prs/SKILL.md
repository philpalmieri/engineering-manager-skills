---
name: enrich-prs
description: |-
  Fetch detailed review data for team-reviewed PRs: review rounds, dismissals,
  approvals, change requests, and per-reviewer actions. Produces enrichment JSON
  files used by metrics skills for quality and cycle analysis. Use when asked to
  'enrich PRs', 'fetch review details', or as part of a data sync.
license: MIT
---

# PR Enrichment

Fetch detailed review timeline data for PRs that received team reviews. This data powers the review-cycles, review-cards, and nine-box metrics skills.

## When to Trigger

- "enrich PRs", "fetch review details", "get PR review data"
- "enrich [month]", "enrich last quarter"
- Called by `team-data-sync` as part of a full data refresh

## Data Sources (Input)

- `data/YYYY-MM/{login}-prs.json` — list of PRs to potentially enrich (need repo and number)
- `data/YYYY-MM/{login}-reviews.json` — identifies which PRs had team reviewers
- `team.json` — team roster (to identify team-internal reviews)

## Data Produced (Output)

- `data/enrichment/{pr_number}.json` — one file per PR with review detail

## Process

### Step 1: Build the enrichment manifest

For each month in the requested range:
1. Load all `{login}-prs.json` files to get authored PRs
2. Load all `{login}-reviews.json` files to identify team reviewers
3. Cross-reference: find PRs where BOTH author AND at least one reviewer are team members
4. Check which of these already have enrichment files in `data/enrichment/`
5. The delta = PRs needing enrichment that don't have a file yet

Report:
```
Enrichment plan for 2026-01 through 2026-03:
  Total team-reviewed PRs: 184
  Already enriched: 142
  Needing enrichment: 42
```

### Step 2: Fetch review timeline for each PR

For each unenriched PR, query the GitHub API for the review timeline:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/reviews \
  --jq '[.[] | {author: .user.login, state: .state, submitted_at: .submitted_at}]'
```

### Step 3: Calculate metrics per PR

From the review timeline, compute:

| Field | Description |
|-------|-------------|
| `team_reviews_count` | Total review events from team members |
| `changes_requested` | Count of CHANGES_REQUESTED events |
| `approvals` | Count of APPROVED events |
| `dismissals` | Count of DISMISSED events |
| `review_rounds` | Number of review cycles (review → commit → review) |
| `final_state` | Last review state (APPROVED, PENDING, CHANGES_REQUESTED) |
| `reviewer_actions` | Per-reviewer breakdown of actions |
| `repo` | Repository (owner/name) |
| `number` | PR number |
| `author` | PR author login |

**Review rounds calculation:**
A "round" = one cycle of: reviewer submits feedback → author pushes commits → reviewer reviews again. Count the number of times this cycle repeats. A PR with no iterations (approved on first review) has `review_rounds: 1`.

### Step 4: Write enrichment file

Save to `data/enrichment/{number}.json`:

```json
{
  "team_reviews_count": 4,
  "changes_requested": 1,
  "approvals": 1,
  "dismissals": 2,
  "review_rounds": 3,
  "final_state": "APPROVED",
  "reviewer_actions": {
    "alice": { "approved": 1, "changes_requested": 0, "commented": 0, "dismissed": 1 },
    "bob": { "approved": 0, "changes_requested": 1, "commented": 0, "dismissed": 1 }
  },
  "repo": "org/repo-name",
  "number": 12345,
  "author": "carol"
}
```

### Step 5: Batch and rate limit

- Process PRs in batches of 10-20 to avoid rate limits
- Pause between batches if rate limit headers indicate throttling
- Report progress: "Enriched 20/42 PRs..."
- If a PR returns 404 (deleted/private), write a minimal stub noting it's inaccessible

### Step 6: Summary report

```
Enrichment complete:
  Processed: 42 PRs
  Successful: 40
  Skipped (404): 2
  Total enrichment files: 184

Coverage by month:
  2026-01: 52/55 PRs enriched (95%)
  2026-02: 61/64 PRs enriched (95%)
  2026-03: 71/65 PRs enriched (100%)
```

## Rules

- **Never overwrite existing enrichment files** unless explicitly asked to "re-enrich" or "refresh"
- **Only enrich team-reviewed PRs** — skip PRs where all reviewers are external
- **Idempotent** — running twice produces the same result (skips already-enriched PRs)
- **Enrichment files are keyed by PR number** — assumes PR numbers are globally unique within the org (they are within a repo; if cross-repo collisions occur, include repo in filename)
- **Rate limit awareness** — check `X-RateLimit-Remaining` header and back off when low

## Notes

- Enrichment data is the most expensive to fetch (1 API call per PR)
- For large teams with high PR volume, a full quarter might be 200-500 API calls
- The `batch-plan.json` file can be used to track progress across interrupted sessions
- PRs older than ~90 days may have review data that's harder to reconstruct from the API
