---
name: weekly-snippet
description: |-
  Draft a weekly status update (snippet) for your manager. Gathers GitHub activity
  for all team members, cross-references vault context and last week's snippet, and
  produces a structured update with wins, upcoming work, and strategic commentary.
  Use when asked to prep or write a weekly snippet.
---

# Weekly Snippet

Draft a weekly status update summarizing team wins, upcoming ships, incidents, and strategic observations. Combines GitHub activity data with vault context to produce a ready-to-edit snippet.

## When to use

- "Prep this week's snippet", "do my snippets", "weekly snippet", "write my update"

## Prerequisites

- Team roster (`team.json`) with member logins
- GitHub CLI (`gh`) authenticated with org access
- Previous snippets in the vault for style reference (location from setup config)
- Obsidian vault with daily notes, project files, and people files

## Process

### Step 1: Calibrate style and continuity

1. Read the most recent snippet file to understand:
   - Current writing style and voice
   - What was listed as "Upcoming Ships" last week (did any ship? → move to Wins)
   - Ongoing threads that need updates
   - Section structure and formatting patterns
2. Optionally read 1-2 older snippets for additional calibration

### Step 2: Scan for snippet ideas in the vault

Search the vault for unchecked tasks tagged `#snippetIdea`:

```bash
grep -rn '\- \[ \].*#snippetIdea' <vault_path>/Dailies/ <vault_path>/Projects/ <vault_path>/People/
```

For each match, read surrounding context. Incorporate into the draft where relevant:
- Strategic/architecture concerns → Top of Mind
- Shipped or in-progress work → Wins or Upcoming Ships
- Track which ideas you used and which you skipped (report after drafting)

### Step 3: Fetch team activity from GitHub

For each team member in the roster, run these searches. Date range: prior Monday through today.

**Authored PRs:**
```bash
gh search prs --author={login} --owner={org} --created={start}..{end} --json number,title,repository,state,createdAt,closedAt --limit 50
```

**Merged PRs (main repo if applicable):**
```bash
gh search prs --author={login} --owner={org} --merged={start}..{end} --json number,title,repository,mergedAt --limit 50
```

**Closed issues (excluding noise):**
```bash
gh search issues --repo={org}/{team-repo} --state=closed --closed={start}..{end} --json number,title,labels,assignees --limit 50
```

Filter out noise (automated issues, monitoring labels, bot-created items) based on team conventions.

**Cap detection:** If any query returns exactly 50 or 100 results, the API likely capped. Split the date range in half and re-query to get complete data.

### Step 4: Cross-reference with last week

- Items from last week's "Upcoming Ships" that now have merged PRs or closed issues → move to Wins
- Items still in progress → keep in Upcoming Ships with updated status
- Don't include items that shipped more than 1 week ago as wins

### Step 5: Draft the snippet

Use this structure:

```markdown
## Weekly Snippet - YYYY-MM-DD

### Top of Mind

**[Topic]**
- Strategic commentary, concerns, observations
- Link relevant issues/PRs

---

### Wins

**[Project Name] ([#issue](url))**
- What shipped, who did it, why it matters
- @mentions for credit

---

### Upcoming Ships

**[Project Name] ([#issue](url))**
- Current status, what's left, target timeline
- Any risks or blockers

---

### Incidents This Week

On-call: @{login}

- **[monitor-name]**: [#issue](url), [date], [status]. Brief description of what happened and resolution.
```

### Step 6: Save and summarize

1. Save to the configured snippets directory with tomorrow's date (or the next business day) as filename: `YYYY-MM-DD.md`
2. Append a live Obsidian Tasks query block at the bottom to surface remaining `#snippetIdea` items:

````markdown
```tasks
not done
description includes #snippetIdea
```
````

3. Summarize what you included: which `#snippetIdea` items were incorporated (and where), which were skipped (and why)
4. Flag anything uncertain (e.g., "not sure if this PR was this week or last")
5. Suggest items the user might want to add manually (meetings, conversations, decisions that don't show up in GitHub)

## Content guidelines

- **Wins should be specific.** Name the person, the PR/issue, and what it does. Not just "worked on X."
- **Top of Mind is for strategic thinking.** Architecture concerns, process observations, cross-team dynamics, things that need attention but aren't tasks.
- **Upcoming Ships should have status.** On track? At risk? Blocked? What's left?
- **Don't repeat old news.** Only include activity from the current week.
- **Credit people.** Use @mentions. Highlight individual contributions.
- **Group related items.** Multiple PRs for the same project go under one heading.
- **Be honest about risks.** If something is slipping, say so. If a dependency is blocking, name it.

## Rules

- **Voice:** Match the user's established snippet style. Read previous snippets before writing.
- **No filler.** Skip projects with no meaningful activity this week.
- **Incidents only if they happened.** Omit the section entirely if no incidents fired.
- **Link everything.** Every project reference should have an issue/PR link for context.
- **Date range matters.** The snippet covers Monday through today. Don't pull in work from before Monday unless it's a carryover from last week's Upcoming.
