---
name: prep-my-day
description: |-
  Full daily prep orchestration for engineering managers. Chains together task rollover,
  1:1 prep for each person on the schedule, and priority focus. Reads today's daily note
  schedule and works through each step sequentially. Use when asked to prep today or
  set up the day.
---

# Prep My Day

Orchestrate a full daily prep by chaining atomic skills in sequence. Reads today's schedule, identifies who needs 1:1 prep, rolls tasks forward, and sets up priorities.

## When to use

- "Prep my day", "prep today", "set up today"

## Prerequisites

- Today's daily note must already exist with a time-blocked schedule
- Team roster (`team.json`) with `relationship` and `vault_name` fields
- GitHub CLI authenticated

## Process

### Phase 1: Read the schedule

1. Read today's daily note
2. Parse the time-blocked schedule to identify 1:1 meetings
3. Match each person to the team roster:
   - `members` with `relationship: "direct"` → use `direct-report-prep` skill
   - `collaborators` with `relationship: "peer"` → use `peer-prep` skill
   - `collaborators` with `relationship: "manager"` → use `peer-prep` skill (with upward topics)
   - Not found → ask the user about the relationship
4. Present the plan:
   > I see 4 meetings today: [Name] (direct, 1:00), [Name] (peer, 1:30), [Name] (direct, 2:00), [Name] (direct, 3:00). I'll prep each one. Ready?

**Skip logic:**
- Non-1:1 meetings (standup, training, interviews): skip
- Already-prepped people (today's date entry exists in their People file): note and skip unless user asks to refresh

### Phase 2: Task rollover

Invoke the `task-rollover` skill:
1. Scan all dailies before today for open tasks
2. Move them to today's daily (with arrows and date suffixes)
3. Mark source tasks as forwarded (`[>]`)
4. Report any escalation-level items (3+ arrows)

### Phase 3: 1:1 prep (sequential, with stops)

Work through each person **one at a time**, in chronological order of meeting time.

For each person:
1. Determine prep type from roster (`direct-report-prep` or `peer-prep`)
2. Invoke the appropriate skill
3. Present summary and ask before moving on:
   > Done with [Name]. Anything to add or adjust before I move to [Next]?

### Phase 4: Scan discussion sources

If the team config has `discussion_sources`, check each for recent posts the user should know about.

1. Fetch discussions created or updated in the last 2-3 business days
2. Filter for relevance:
   - Announcements, policy changes, or pinned posts
   - Posts from related leadership or partner teams
   - Posts with high engagement (many comments/reactions)
   - Topics relevant to the team's domain
3. Surface relevant posts as `#today` tasks on today's daily:
   ```markdown
   - [ ] Read: [Discussion title](url) — 1-line summary of why it's relevant #today
   ```
4. Skip: routine posts, unrelated team announcements, recurring meeting notes

Place in the `📥 Moved Open Tasks` section or a dedicated `📣 Discussions` section if there are multiple items.

### Phase 5: Today's priorities

Invoke the `today-priorities` skill:
1. Scan vault for high-priority open items
2. Check for time-sensitive deadlines
3. Present grouped by urgency
4. Write Priority Focus section to today's daily
5. Tag confirmed items with `#today`

### Phase 6: Summary

After all phases:
> Day prepped:
> - Rolled N tasks forward (M flagged for escalation)
> - Prepped 1:1s: [Name] (3 topics), [Name] (2 topics), ...
> - N priority items tagged for today
>
> Anything else before you start?

## Fast path

If the user says "just do it all", "no stops", or "fast mode":
- Skip per-person confirmations
- Write all prep sequentially
- Present a single summary at the end

## Interaction model

Default is **sequential with stops** between each person. This lets the user add context or corrections as you go.

The user can switch to fast path at any point: "Just do the rest, no stops."
