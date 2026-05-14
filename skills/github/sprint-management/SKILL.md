---
name: sprint-management
description: |-
  Automate sprint lifecycle: start a new sprint (create iteration, roll over incomplete
  items), end a sprint (snapshot results, capture metrics), and manage the transition
  between sprints. Use at sprint boundaries or when setting up a new iteration.
---

# Sprint Management

Automate the sprint lifecycle on GitHub Projects. Handles creating iterations, rolling over incomplete work, snapshotting results, and keeping the board clean between sprints.

## When to use

- "Start the next sprint", "end this sprint", "roll over sprint items"
- At the beginning or end of a sprint cycle
- When incomplete work needs to move to the next iteration

## Prerequisites

- GitHub Projects board with iteration field configured
- GitHub CLI (`gh`) authenticated with org and project access
- Team roster (`team.json`) for context on assignees

## Operations

### Start Sprint

Triggered by: "start sprint", "kick off next sprint", "new sprint"

#### Step 1: Detect sprint state

```bash
# Get iteration field metadata
gh project field-list {project-number} --owner {org} --format json
```

Identify:
- Current iteration (date range includes today, or most recently ended)
- Next iteration (if it already exists)
- Whether there's a gap or overlap

If the next iteration doesn't exist yet, create it:
```bash
gh project field-create {project-number} --owner {org} \
  --name "Sprint [N+1]" --data-type ITERATION
```

If using the GitHub Projects iteration field (which auto-creates iterations based on duration), just identify which iteration is "next."

#### Step 2: Roll over incomplete items

Find items from the previous sprint that aren't done:

```bash
# Get all items in the project
gh project item-list {project-number} --owner {org} --format json --limit 200
```

Filter to items where:
- Sprint iteration = previous sprint
- Status != "Done" (or whatever the team's completion column is)

For each incomplete item:

1. **Update the iteration field** to the new sprint
2. **Add a comment** noting the rollover:
   ```
   Rolled over from Sprint N → Sprint N+1. [Originally planned for {date range}]
   ```
3. **Track the rollover** for the summary

#### Step 3: Set sprint goals

Ask the user what the sprint goals are, or detect them from:
- Issues labeled with initiative/epic labels assigned to the new sprint
- Items marked as high priority in the new iteration

Present the proposed sprint composition:

```
Sprint N+1 (May 19 → Jun 1):

Rolled over (5 items):
  ⚠️ #1236 API endpoint (was In Progress)
  ⚠️ #1238 E2E tests (was Todo, blocked)
  ...

New items already assigned (8 items):
  #1300 New feature X
  #1301 Bug fix Y
  ...

Sprint goals (epics with items in this sprint):
  1. Notification Model (#1234) — 2 remaining items
  2. Dashboard Redesign (#1290) — 4 new items
  ...
```

Ask: "Does this look right? Anything to add, remove, or re-prioritize?"

#### Step 4: Confirm and save

After user confirmation:
- Ensure all items have correct iteration assignment
- Update status of rolled-over items if needed (e.g., reset "In Review" back to "In Progress" if the PR was abandoned)
- Generate a sprint kickoff summary

### End Sprint

Triggered by: "end sprint", "close this sprint", "sprint wrap-up"

#### Step 1: Snapshot the sprint

Fetch all items assigned to the current sprint:

```bash
gh project item-list {project-number} --owner {org} --format json --limit 200
```

Categorize by final status:

| Category | Criteria |
|----------|----------|
| ✅ Completed | Status = Done, issue closed, PR merged |
| 🔄 Incomplete | Status != Done, will roll over |
| ❌ Dropped | Items removed from sprint mid-cycle (if trackable) |
| ➕ Added | Items added after sprint start (scope creep signal) |

#### Step 2: Capture metrics

Calculate:
- **Completion rate**: done / total planned (exclude mid-sprint additions)
- **Scope change**: items added or removed mid-sprint
- **Goal completion**: for each sprint goal, did all child issues ship?
- **Rollover count**: how many items are carrying over
- **Per-person breakdown**: items completed by assignee

#### Step 3: Generate sprint summary

```markdown
## Sprint [N] Summary: [Start] → [End]

### Results
- **Planned:** [X] items across [Y] goals
- **Completed:** [Z] items ([%] completion rate)
- **Rolling over:** [N] items
- **Added mid-sprint:** [M] items

### Goal Status
| Goal | Items | Done | Status |
|------|:-----:|:----:|--------|
| [Epic 1] | 6 | 6 | ✅ Complete |
| [Epic 2] | 4 | 2 | ⚠️ Partial (rolling 2) |

### By Team Member
| Member | Planned | Completed | Rolled |
|--------|:-------:|:---------:|:------:|
| @alice | 4 | 3 | 1 |
| @bob | 3 | 3 | 0 |

### Items Rolling Over
| Issue | Assignee | Status | Reason |
|-------|----------|--------|--------|
| [#1236](url) | @bob | In Progress | Race condition found late |

### Observations
- [Patterns, risks, process observations for retro discussion]
```

#### Step 4: Transition

After the user reviews:
- Save the summary to the vault or as a GitHub discussion
- Prompt: "Ready to start Sprint N+1? I'll roll over the incomplete items."
- If yes, chain into the Start Sprint flow

### Rollover Only

Triggered by: "roll over sprint items", "move incomplete to next sprint"

A lightweight version of the full start/end cycle. Just moves incomplete items from the most recently ended sprint to the current/next one without the full snapshot and metrics.

## Sprint detection logic

Since teams configure iterations differently:

1. **Iteration field exists**: Use `gh project field-list` to find iteration fields, parse their configuration for start/end dates
2. **Date-based**: If iterations have date ranges, match today's date to find current
3. **Manual**: If iterations are just named (e.g., "Sprint 14"), ask the user which is current
4. **Fallback**: Ask the user for the sprint name/number and date range

## Rules

- **Don't auto-modify without confirmation.** Always present the plan (rollovers, new items, goals) and get a "yes" before making changes to the project board.
- **Comment on rolled items.** When moving an item to the next sprint, add a comment so the history is visible. Teams need to know something slipped.
- **Preserve status.** When rolling over, keep the item's current status (In Progress stays In Progress). Only reset if the user asks.
- **Track scope creep.** Items added mid-sprint should be flagged in the summary. This isn't judgment; it's data for retro.
- **Sprint goals are the frame.** Everything rolls up to goals. Individual item counts are secondary to "did we hit what we committed to?"
- **Fiscal year awareness.** If the team uses a custom fiscal calendar, respect that mapping from the team roster config for quarterly sprint numbering.
