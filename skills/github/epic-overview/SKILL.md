---
name: epic-overview
description: |-
  Generate a grouped overview of all active epics and initiatives your team owns,
  organized by strategic objective or project area. Shows what each initiative is,
  which epics roll up to it, who owns them, progress, and flags unassigned or
  stale work. Default output is conversational; optionally save to vault or file.
---

# Epic Overview

Produce a structured view of all active epics and initiatives, grouped by their parent objective or project area. Each group gets a short description of what it is, then lists every epic under it with owner, progress, and a one-liner explanation.

## When to use

- "Epic overview", "show me all our epics", "what initiatives are we running"
- "Map our work", "project portfolio", "what does the team own"
- Onboarding context: "give someone a picture of everything we're working on"

## Prerequisites

- GitHub CLI (`gh`) authenticated with org access
- Team repo with issues labeled `epic`, `initiative`, or `batch`
- Team roster (`team.json`) for DRI identification

## Output

Default: conversational (printed to screen). User can ask to save it to a vault note, markdown file, or paste destination.

## Process

### Step 1: Gather all open epics and parent issues

Query the team repo for:
```
- All open issues with label: epic, initiative, or batch
- All open issues with sub_issues_summary.total > 0 (acting as parent issues)
```

For each, capture: number, title, assignees, labels, updated_at, sub_issues progress.

### Step 2: Identify DRI

DRI = first assignee on the issue. If no assignee, mark as "unassigned."

### Step 3: Group by initiative/objective

Analyze the epics by:
- Labels (e.g., `virtual-registry`, `authentic-contributions`, `Releases`)
- Title patterns and naming conventions
- Parent-child relationships (sub-issues of larger epics)
- Shared assignees and project area context

Create logical groupings that represent strategic objectives or project areas. Each group should have:
- A meaningful name (not just a label echo)
- A 1-2 sentence description of what the objective is

Common grouping patterns:
- Product feature area (e.g., "Releases & Immutable Releases")
- Customer integration (e.g., "MDC / Virtual Registry")
- Infrastructure domain (e.g., "Signing Infrastructure")
- Operational category (e.g., "Maintenance & Reliability")

### Step 4: Write the overview

For each group:

```markdown
## [Initiative Name]

[1-2 sentence description of what this initiative is and why it matters.]

- [#N Title](url) — @dri
  [One sentence: what this epic actually does.] Progress: X/Y (Z%).
- [#N Title](url) — unassigned
  [Description.] Progress: X/Y (Z%).
```

### Step 5: Surface unknowns

At the end, include a section for:
- Epics that don't clearly fit any group
- Stale epics (no activity 60+ days) that may be dead
- Completed epics (100% or all sub-issues closed) that should be closed

### Step 6: Output

**Default:** Print to screen as markdown.

**If user asks to save:** Write to specified location (e.g., vault note, project file). Include a date header so it's clear when the overview was generated.

## Content guidelines

- **One sentence per epic.** The overview is for scanning, not deep reading. Just say what it does.
- **Group by what makes sense, not by label.** Sometimes one label covers multiple objectives; sometimes multiple labels are really one thing.
- **Show progress honestly.** X/Y sub-issues with percentage. If 0/0, say "no sub-issues tracked."
- **Flag unassigned work.** Every epic should have a DRI. Unassigned = a decision to make.
- **Include everything open.** Don't filter by "active" here. The point is the complete picture, including work that's stalled or forgotten.
- **Dead epics go at the bottom.** Don't bury them in their group where they clutter the active work view.

## Relationship to other skills

- **epic-cleanup:** Cleanup is the actionable version (tasks to assign, close, decide). Overview is the reference map.
- **project-status:** Status covers the last 7 days of movement. Overview is the static "what exists" view regardless of recent activity.
- **sprint-status:** Sprint is scoped to the current iteration. Overview is all open work.
