---
name: epic-cleanup
description: |-
  Audit all active epics for ownership gaps, stale work, and closure candidates.
  Produces an actionable checklist: assign DRIs, close completed/dead epics,
  and make decisions on ambiguous work. Organized by person then by action type.
  Default output is conversational; optionally save to vault or file.
---

# Epic Cleanup

Audit all active epics and initiatives for ownership gaps, staleness, and hygiene issues. Produces a person-by-person ownership map plus actionable task lists for assigning, closing, and deciding on epics.

## When to use

- "Epic cleanup", "audit our epics", "who owns what"
- "Which epics need a DRI", "what should we close"
- Sprint planning: "clean up the board before we start"
- Quarterly: "are all our epics still relevant"

## Prerequisites

- GitHub CLI (`gh`) authenticated with org access
- Team repo with issues labeled `epic`, `initiative`, or `batch`
- Team roster (`team.json`) for member identification

## Output

Default: conversational (printed to screen). User can ask to save it as a vault note with task checkboxes for tracking.

## Process

### Step 1: Gather all open epics and parent issues

Same as epic-overview: query for all open issues that are epics, initiatives, batches, or have sub-issues.

### Step 2: Build ownership map

For each team member, list every epic/initiative where they are assigned (DRI). Include:
- Issue number and title (linked)
- Progress (sub-issues completed / total)
- Activity status: ✅ Active, ⚠️ Stale (14+ days), ❓ No signal (30+ days)

### Step 3: Identify action items

Categorize into:

**Assign immediately** — Epics with active work happening (sprint goals, recent PRs) but no DRI assigned.

**Assign or close (nearly done)** — Epics at 80%+ completion with no DRI to close out the remaining items.

**Probably close** — Epics that are:
- 100% complete but still open
- Superseded by newer work
- Stale for 60+ days with no indication of resuming
- From a previous fiscal year with no recent relevance

**Needs a decision** — Epics where it's unclear if the work is still planned, deprioritized, or abandoned. Surface these for the manager to make a call.

**Unassigned parent issues** — Issues acting as epics (have sub-issues) but no one is assigned. Need a DRI decision.

### Step 4: Write the output

#### Person-by-person section:

```markdown
### @login (Name)

- [#N Title](url) — X/Y (Z%) status_emoji description
- [#N Title](url) — X/Y (Z%) status_emoji description
```

#### Action items section (when saving to vault, use checkboxes):

```markdown
## Suggested Actions

**Assign immediately:**
- [ ] [#N](url) Assign to [[Person]] (reason)

**Assign or close:**
- [ ] [#N](url) Title (X%) — who finishes this?

**Probably close:**
- [ ] [#N](url) Title (reason it's dead)

**Needs a decision:**
- [ ] [#N](url) Title — still planned or deprioritized?

**Unassigned:**
- [ ] [#N](url) Title (X/Y) — suggested owner?
```

### Step 5: Output

**Default:** Print to screen as markdown (no checkboxes, just bullets with recommendations).

**If user asks to save to vault:** Use `- [ ]` task checkboxes so items are trackable. Include wikilinks to People files for assignee suggestions. Link all issue numbers.

## Staleness thresholds

| Days since last activity | Signal |
|---|---|
| 0-14 | ✅ Active |
| 14-30 | ⚠️ Getting stale |
| 30-60 | ❓ No signal, needs check-in |
| 60+ | 🔴 Likely dead or deprioritized |

Activity = any of: comment posted, sub-issue closed, PR merged referencing the epic, label change, assignee change.

## Content guidelines

- **Be direct about recommendations.** Don't just flag "unassigned" — suggest who should own it based on their existing work.
- **Group actions by effort level.** "Assign immediately" is 30 seconds of work. "Needs a decision" requires a conversation.
- **Use wikilinks when saving to vault.** Link People files so the user can navigate to context.
- **Link every issue.** Full URLs so items are one click away.
- **Call out load imbalance.** If one person has 8 epics and another has 0, say so.
- **Don't mix cleanup with status.** This isn't about "how is work going" — it's "is our board accurate and owned."

## Relationship to other skills

- **epic-overview:** Overview is the "what exists" reference. Cleanup is the "what needs fixing" action list.
- **project-status:** Status reports on weekly progress. Cleanup is periodic hygiene (monthly or quarterly).
- **sprint-management:** Sprint management moves items between sprints. Cleanup ensures the backlog of epics is healthy.
