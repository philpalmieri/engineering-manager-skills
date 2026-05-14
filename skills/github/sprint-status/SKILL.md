---
name: sprint-status
description: |-
  Analyze current sprint health by examining sprint goals, issue status, PR readiness,
  and recent comments for risks and blockers. Produces a goal-oriented sprint health
  report with confidence assessments. Use during check-ins, refinement, or async updates.
---

# Sprint Status

Analyze the current sprint against its goals. Goes beyond "what's in progress" to assess whether sprint goals will actually be met, surface risks from issue/PR activity, and flag items that need attention.

## When to use

- **"What is the current sprint?"** — Quick summary: name, dates, issue counts, goals, on track/at risk
- **"Get sprint breakdown"** — Full analysis: goal-by-goal status, PR readiness, comment signals, risk assessment
- **"Update sprint status"** — Generate a copyable status update to paste into the project board's status/notes field
- Before backlog refinement or team check-ins
- Mid-sprint risk assessment
- As input for weekly snippets

## Prerequisites

- GitHub Projects board with sprint iterations configured
- Sprint goals defined as epics or initiative issues assigned to the current iteration
- GitHub CLI (`gh`) authenticated with org and project access
- Team roster (`team.json`) for mapping assignees

## Modes

This skill operates in three modes depending on what the user asks for. All three share Steps 1-2 (identify sprint, fetch items). They diverge after that.

### Mode 1: Current Sprint (Quick Summary)

Triggered by: "what is the current sprint?", "current sprint", "where are we?"

A fast, lightweight answer. No deep analysis. Just the facts.

**Output:**
```markdown
## Sprint [Name] ([Start] → [End], [N] days remaining)

**Goals:**
1. [Epic/Initiative title] — [X/Y] issues done
2. [Epic/Initiative title] — [X/Y] issues done

**Counts:** [Done] completed, [In Progress] in progress, [Todo] not started, [Total] total

**Status:** [On Track / At Risk / Behind] — [one-sentence justification based on completion % vs. time elapsed]
```

The on-track/at-risk assessment here is real, not a label someone set on the board. Compare completion percentage against time elapsed: if 30% done with 20% time remaining, that's at risk.

After the quick summary, skip to Step 7 (present to user).

### Mode 2: Sprint Breakdown (Full Analysis)

Triggered by: "get sprint breakdown", "sprint breakdown", "sprint health check", "how's the sprint going?"

This is the full analysis. Run all steps (1-7) as documented below.

### Mode 3: Update Sprint Status (Pasteable Note)

Triggered by: "update sprint status", "write sprint update", "sprint status note"

Generate a concise, pasteable status update designed to be added to the project board's status/notes field. This is what the user posts for async visibility.

**Process:**
1. Run Steps 1-5 (identify sprint, fetch items, group by goal, analyze signals, assess confidence)
2. Read any existing status notes/updates on the project board to understand the update cadence and voice
3. Generate the update in this format:

```markdown
**Sprint [Name] Update — [Date]**

[1-2 sentence overall status. Direct and honest.]

**Goals:**
- ✅ [Goal 1]: [one-line status, what shipped]
- ⚠️ [Goal 2]: [one-line status, what's at risk and why]
- 🔄 [Goal 3]: [one-line status, in progress]

**Risks:**
- [Risk 1 with context, not just "at risk"]

**Wins:**
- [Notable completions this sprint]

**Next up:**
- [What's being worked on in the remaining days]
```

Present this as a copyable block. The user will paste it into the project board.

---

## Process (Full Breakdown)

### Step 1: Identify the current sprint

Query the project board to find the active sprint iteration:

```bash
# List project iterations to find current one
gh project field-list {project-number} --owner {org} --format json
```

Parse iteration fields to find the one whose date range includes today. Extract:
- Sprint name/number
- Start date and end date
- Days remaining

If you can't determine the current sprint programmatically, ask the user.

### Step 2: Fetch sprint items

```bash
# Get all items in the project with their field values
gh project item-list {project-number} --owner {org} --format json --limit 200
```

Filter to items assigned to the current sprint iteration. For each item, extract:
- Issue/PR number and title
- Status column (Todo, In Progress, In Review, Done)
- Assignee(s)
- Labels (especially epic/initiative labels)
- Parent issue (if applicable)

### Step 3: Group by sprint goal

Sprint goals are the epics or initiative-level issues in the sprint. Group child issues under their parent:

```
Sprint Goal: "Notification Model for XYZ" (#1234)
├── ✅ #1235 Backend data model (Done)
├── 🔄 #1236 API endpoint (In Progress, PR open)
├── 🔄 #1237 Frontend integration (In Review)
└── ⬜ #1238 E2E tests (Todo)
```

Items without a parent get grouped under "Ungrouped / Standalone."

### Step 4: Analyze activity signals

For each sprint goal and its child issues, fetch recent activity:

**Issue comments (within sprint window):**
```bash
gh issue view {number} --repo {org}/{repo} --json comments
```

Scan comments for signal:
- **Blockers**: "blocked by", "waiting on", "can't proceed", "dependency"
- **Risks**: "found an issue", "more complex than expected", "scope change", "need to revisit"
- **Wins**: "shipped", "merged", "done", "looks good"
- **Updates**: DRI status updates, decision notes

Also check the parent/epic issue for updates posted during the sprint window.

**Open PR analysis:**
```bash
gh pr view {number} --repo {org}/{repo} --json state,reviewDecision,reviews,comments,additions,deletions,createdAt,updatedAt
```

For each open PR, assess:
- **Review status**: approved, changes requested, no reviews yet
- **Staleness**: days since last update (>3 days in a 2-week sprint = concern)
- **Size**: large PRs (>500 lines) late in sprint = risk
- **Comment sentiment**: active discussion vs. silence vs. unresolved threads

### Step 5: Assess confidence per goal

For each sprint goal, synthesize a confidence rating:

| Rating | Criteria |
|--------|----------|
| ✅ **On Track** | >50% child issues done, remaining have active PRs with approvals, no blockers in comments |
| ⚠️ **At Risk** | Open items with stale PRs, blocker language in comments, large scope remaining with <40% sprint left |
| 🔴 **Blocked** | Explicit blocker identified, dependency not met, no PR activity in 3+ days on critical path items |
| ✅ **Complete** | All child issues closed/merged |

Include a brief justification for each rating. Reference specific comments, PR states, or patterns that informed the assessment.

**Confidence note:** When inferring risk from PR activity or comment sentiment, state your confidence level. "PR #456 has been open 5 days with no reviews; this is likely at risk" is better than "this will miss the sprint."

### Step 6: Generate the report

```markdown
## Sprint Status: [Sprint Name] ([Start] → [End], [N] days remaining)

### Summary
- [X] of [Y] sprint goals on track
- [N] items completed, [M] in progress, [K] not started
- [Risks/blockers count] items flagged

---

### [Sprint Goal 1]: [Title] ([#epic](url))
**Status: ⚠️ At Risk**
**DRI:** @{assignee}

| Issue | Status | Assignee | Notes |
|-------|--------|----------|-------|
| [#1235](url) Backend model | ✅ Done | @alice | Merged 5/10 |
| [#1236](url) API endpoint | 🔄 In Progress | @bob | PR open, 1 approval, 1 change request |
| [#1237](url) Frontend | 🔄 In Review | @carol | Large PR (800 lines), opened yesterday |
| [#1238](url) E2E tests | ⬜ Not started | @bob | Depends on #1236 |

**Activity signals:**
- @bob commented on #1236 (5/12): "found a race condition in the job queue, investigating"
- PR #1236 has unresolved review thread about error handling
- #1238 can't start until #1236 merges; with 4 days left, this is tight

**Assessment:** API endpoint has a known issue being investigated. E2E tests are blocked behind it. If the race condition fix takes more than a day, tests won't make this sprint.

---

### [Sprint Goal 2]: [Title] ([#epic](url))
**Status: ✅ On Track**
...

---

### Ungrouped Items
[Items in the sprint without a parent epic]

---

### Risks & Blockers
1. 🔴 [Specific blocker with context and suggested action]
2. ⚠️ [Risk with mitigation suggestion]

### Wins This Sprint
- [Notable completions worth calling out]
```

### Step 7: Present to user

Summarize the key findings conversationally:

> Sprint is 60% through. 2 of 4 goals on track, 1 at risk (API endpoint has a race condition, blocking tests), 1 complete. 3 PRs need review attention. Want me to save the full report?

## Rules

- **Goals over items.** The sprint succeeds or fails at the goal level, not the individual issue level. Always frame status in terms of "will this goal be met?"
- **Evidence-based assessments.** Every confidence rating should cite specific data (PR state, comment, date, staleness). Don't rate on vibes.
- **Distinguish inference from fact.** "PR has been open 5 days" is fact. "This probably won't merge in time" is inference. Label them differently.
- **Don't catastrophize.** A stale PR might just mean someone was on PTO. Note the risk but don't assume the worst.
- **Actionable flags.** Every risk or blocker should come with a suggested action: "needs review", "assignee should update", "scope cut candidate", "escalate dependency."
- **Respect the sprint boundary.** Only analyze items assigned to the current sprint. Don't pull in backlog items or future sprint work.
