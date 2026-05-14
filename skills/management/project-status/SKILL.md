---
name: project-status
description: |-
  Generate a weekly Project Status Report covering all active epics and initiatives
  your team owns. Infers status from actual signals (PRs, issue closures, comments,
  board movement) rather than requiring manual DRI updates. Surfaces wins, risks,
  stale projects, and the week ahead. Best run Monday mornings.
---

# Project Status Report

Generate a signal-based project status report by reading actual GitHub activity across all active epics and initiatives your team touches. No manual updates required from DRIs; status is inferred from PRs merged, issues closed, comments posted, and board state changes.

## When to use

- Monday mornings: "project status", "what's the state of everything"
- Start of sprint: "sprint kickoff status"
- Ad hoc: "where do things stand", "project health check"

## Prerequisites

- Team roster (`team.json`) with member logins and project assignments
- GitHub CLI (`gh`) authenticated with org access
- Obsidian vault for output storage
- Access to the team's GitHub Project board (project number in `team.json`)

## Output

A vault note saved to a configurable location (e.g., `Projects/Status Reports/YYYY-MM-DD.md`) plus a conversational summary highlighting items that need attention.

## Process

### Step 1: Identify all active epics and initiatives

Query the team's issue tracker for:
- All open issues labeled `epic`, `initiative`, or `batch` in the team repo
- All open issues with sub-issues (parent issues acting as epics)
- Issues assigned to any team member that have child tasks

Build the canonical list of "things the team is responsible for." This list should be stable week-to-week (projects don't appear/disappear frequently).

### Step 2: Gather activity signals (last 7 days)

For each team member:
- **PRs authored:** merged and open, with repo and title
- **PRs reviewed:** to understand cross-pollination
- **Issues closed:** in the team repo
- **Comments on epics:** any DRI-posted status updates

For each epic/initiative:
- **Sub-issue movement:** how many sub-issues changed status (opened, closed, moved columns)
- **PR activity:** PRs that reference the epic or its sub-issues
- **Comments:** any status updates, decisions, or blockers posted
- **Board status:** current column (Planned, Doing, In Review, Done, Blocked)
- **Trending field:** if the project board has a trending/health indicator, read it

### Step 3: Infer status per project

For each epic, assign an inferred status:

| Signal | Inference |
|--------|-----------|
| PRs merged + issues closed this week | 🟢 Active, making progress |
| PRs open but not merged, reviews pending | 🟡 In flight, waiting on reviews |
| Sub-issues moved to Doing/In Review | 🟡 Work started, not yet landed |
| No activity, but sprint just started | ⚪ Not started yet (OK if early sprint) |
| No activity for 7+ days on a "Doing" epic | 🔴 Stale, needs attention |
| No activity for 14+ days | 🔴 No signal, flag for check-in |
| Blocked label or blocker comments | 🔴 Blocked, surface the blocker |

### Step 4: Categorize into report sections

Group findings into:

**Wins (last 7 days)**
- Epics or milestones completed
- Significant PRs merged that close out sprint goals
- Anything that moved from "In Progress" to "Done"

**In Flight**
- Active work with PRs open or issues moving
- Note who's doing what and expected landing

**At Risk**
- Projects with no activity that should have some
- Things approaching deadlines with incomplete work
- Blocked items with no resolution path visible

**No Signal**
- Epics assigned to the team with zero activity in the window
- Not necessarily bad (could be deprioritized), but worth acknowledging

**Week Ahead**
- Items in "Planned" that should start this week
- Upcoming deadlines (compliance, a11y audits, customer commitments)
- Anything from the sprint goals not yet on the board

### Step 5: Write the report

#### Report format

```markdown
## Project Status — YYYY-MM-DD

**Sprint:** [Sprint name] (Day N of 14)
**Report window:** [start date] → [end date]

---

### 🟢 Wins

**[Project Name]** ([#epic](url)) — @dri
- What shipped, what closed, what's done
- Specific PRs/issues with links

---

### 🟡 In Flight

**[Project Name]** ([#epic](url)) — @dri
- Current state: what's open, what's being reviewed
- Expected: when it should land

---

### 🔴 At Risk

**[Project Name]** ([#epic](url)) — @dri
- Why it's flagged: no activity / approaching deadline / blocked
- Last activity: [date]
- Suggested action: [what needs to happen]

---

### ⚪ No Signal

**[Project Name]** ([#epic](url)) — @dri
- Last activity: [date]
- Note: [deprioritized? waiting on external? just quiet?]

---

### 📅 Week Ahead

- [ ] [Thing that should start or land this week]
- [ ] [Deadline approaching]
- [ ] [Sprint goal not yet on board]

---

### DRI Update Tracker

| Person | Projects Owned | Updated This Week | Stale |
|--------|:-:|:-:|:-:|
| @name | N | N | N |
```

### Step 6: Save and summarize

1. Save the full report to the vault
2. Present a brief conversational summary (5-8 lines) highlighting:
   - Number of projects on track vs at risk
   - Top 2-3 things that need your attention this week
   - Any patterns (e.g., "nobody updated anything", "Brian is carrying 4 projects")

## Content guidelines

- **Infer, don't require.** The whole point is that DRIs don't need to manually post updates; you read the signals and write the status.
- **Be honest about unknowns.** If there's no signal, say "no signal" rather than guessing.
- **Keep it scannable.** Bold project names, use emoji status indicators, keep descriptions to 1-2 lines per project.
- **Credit people.** Name who shipped what.
- **Flag load imbalance.** If one person is DRI on 5 things and another on 0, note it.
- **Don't repeat old news.** Only wins from the last 7 days. Prior wins live in older reports.
- **Link everything.** Epic numbers, PR numbers, issue numbers. Make it clickable.

## Relationship to other skills

- **weekly-snippet:** The snippet is YOUR update to YOUR manager. This report is the team-wide project health view that feeds into the snippet.
- **sprint-status:** Sprint status is focused on the current sprint board. This covers ALL active work, including things not in the sprint.
- **team-activity:** Team activity is raw data (PRs, reviews, issues). This is the analyzed, narrativized version.

## Cadence

Best used Monday mornings as a "state of the world" before the week starts. Can also be run mid-sprint or before leadership syncs. The Monday framing lets you ask: "what landed last week, and what needs to happen this week?"

## Configuration

The skill needs to know:
- **Team repo:** where epics/issues live (e.g., `org/team-repo`)
- **Project board number:** for sprint/status field data
- **Vault output path:** where reports are saved
- **Epic labels:** what labels identify epics in your repo (default: `epic`, `initiative`, `batch`)
- **DRI field:** how DRI is determined (default: issue assignee)
