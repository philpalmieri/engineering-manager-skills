---
name: peer-prep
description: |-
  Prepare partnership-oriented 1:1 talking points for a peer, manager, or cross-functional
  collaborator. Focuses on shared alignment, open commitments, and strategic topics.
  Use when prepping for a 1:1 with someone you do NOT manage.
license: MIT
---

# Peer / Non-Direct 1:1 Prep

Prepare talking points for a 1:1 with a peer, manager, or cross-functional partner. Focuses on partnership, shared alignment, and open commitments rather than coaching.

## When to use

- "Prep 1:1 with [peer name]", "peer 1:1", "talking points for [name]"
- As part of the `prep-my-day` orchestration skill (for non-direct-report 1:1s)

## Key difference from direct report prep

This is NOT coaching prep. Do not:
- Analyze their code or PR quality
- Frame topics as coaching opportunities
- Use language that implies a reporting relationship

Instead focus on:
- Shared project alignment and partnership
- Open action items between the two of you
- Strategic discussion topics
- Things you need FROM this person or want to ALIGN ON

## Prerequisites

- Team roster with the person listed under `collaborators`
- A People file for the person in the vault
- `relationship` field: `"peer"` or `"manager"`

## Process

### Step 1: Identify the person

1. Match to team roster `collaborators` array
2. If the person is in `members` with `"relationship": "direct"`, redirect to the direct-report-prep skill
3. If not found, ask the user about the relationship

### Step 2: Gather data

**From the vault:**
- Read `People/{vault_name}.md` for:
  - Open items and recent conversation notes
  - Pending asks or commitments ("they were going to [do X]")
  - Items tagged `#followup`

**From recent dailies (last 5-7 days):**
- Mentions of the person
- Shared project context
- Decisions or commitments made

**From project files:**
- Check active projects for references to this person
- Cross-reference with current focus areas for alignment topics

**For manager-relationship people:**
- Surface items you may want to escalate or get guidance on
- Check for blockers, resource asks, or strategic alignment topics

### Step 3: Build talking points

**Open follow-ups** — Commitments either person made. Frame as "did this happen?" not "why didn't you do this?"

**Shared project alignment** — Active projects you're both involved in. What decisions need to be made? Where might you be misaligned?

**Partnership rhythms** — Process questions: cadence, async communication, review workflows.

**Strategic topics** — Bigger-picture items: roadmap, cross-team dynamics, organizational context.

**Reflections/feedback season** — If it's feedback season, consider whether there are themes to discuss conversationally (but only if the user has flagged this or it's contextually relevant).

### Step 4: Write to People file

Same format and rules as direct-report-prep:

```markdown
#### [[Dailies/YYYY-MM-DD|YYYY-MM-DD]] 1:1

**Topic** (Follow-up)
- Follow up: context about the open item

**Topic** (Alignment)
- Ask: strategic question about shared work
```

**Context types:** `(Follow-up)`, `(Alignment)`, `(Partnership)`, `(Strategic)`, `(Escalation)` (manager only)

## Rules

- Same task/formatting rules as direct-report-prep (no new checkboxes, wikilinks, idempotent)
- **Tone matters.** This is a peer conversation. No coaching language, no implied hierarchy.
- If the person's relationship is `"manager"`, include upward topics: escalations, resource asks, guidance needs.
