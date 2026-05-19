---
name: direct-report-prep
description: |-
  Prepare coaching-oriented 1:1 talking points for a direct report. Gathers GitHub
  activity, vault context, and open follow-ups to build structured prep notes.
  Use when prepping for a 1:1 with someone you manage.
license: MIT
---

# Direct Report 1:1 Prep

Prepare talking points for a 1:1 with a direct report. Combines GitHub activity data with vault context (People file, recent dailies) to generate coaching-oriented prep.

## When to use

- "Prep 1:1 for [name]", "prep directs", "1:1 prep for [name]"
- As part of the `prep-my-day` orchestration skill

## Prerequisites

- Team roster (`team.json`) with the person listed under `members` with `relationship: "direct"`
- A People file for the person in the vault's `People/` directory
- GitHub CLI (`gh`) authenticated with access to the org

## Process

### Step 1: Identify the person

1. Match the requested name to a member in the team roster using `name`, `login`, or `vault_name`
2. Confirm the person has `"relationship": "direct"`
3. If ambiguous (multiple matches), ask the user to clarify

### Step 2: Gather data

**From the vault:**
- Read `People/{vault_name}.md` for:
  - Open todos (`- [ ]`) from previous entries
  - Recent 1:1 notes and recurring themes
  - Items tagged `#feedback` or `#followup`
  - Stale items (open tasks with dates older than 2 weeks)
  - Items with `⏫` or `📅` markers (high priority or time-sensitive)

**From GitHub (last 7 days):**
```bash
gh search prs --author={login} --org={org} --created={7_days_ago}..{today} --json number,title,repository,state,mergedAt
gh search prs --author={login} --org={org} --merged={7_days_ago}..{today} --json number,title,repository,mergedAt
```

Extract: PR number, title, repo, state, merged date. Note volume and breadth.

**From recent dailies:**
- Check last 5-7 daily notes (both active and archive) for:
  - Mentions of the person's name or vault link
  - Open action items involving them
  - Context from previous discussions

### Step 3: Build talking points

Organize into these categories:

**Recognition** — Always lead with something positive if there's signal. Name specific PRs, review quality, initiative shown. Don't be generic ("great work this week"). Be specific ("the refactor in PR #1234 was clean, especially the test coverage").

**Open follow-ups** — Reference existing open tasks from prior entries that are still incomplete. Mention when they were first noted. Do NOT recreate them as new `- [ ]` checkboxes (the People file's DataviewJS query already surfaces open tasks).

**Project status** — For active projects the person is working on, check PR activity and note current state. Frame as check-in questions, not status demands. "How's the migration going?" not "Why isn't the migration done?"

**Coaching opportunities** — If a PR or pattern suggests a growth area, frame it as a question. "The refactor PR is solid; have you thought about how to test the migration path?" Not "You should test the migration path."

**Feedback items** — Any `#feedback` tagged items should be surfaced with context.

### Step 4: Write to People file

Write directly into `People/{vault_name}.md`, prepended before the most recent existing entry.

**Format:**
```markdown
#### [[Dailies/YYYY-MM-DD|YYYY-MM-DD]] 1:1

**Topic Name** (Context Type)
- Brief context or PR links
- Check in: question about the topic

**Another Topic** (Recognition)
- What they shipped and why it matters
- Call out: specific good work here
```

**Context types:** `(Recognition)`, `(Follow-up)`, `(Project Status)`, `(Coaching)`, `(Feedback)`

### Step 5: Present summary

Before writing (unless the user said "just do it"), present a brief summary:

> For [Name]: found N open PRs on [project], M stale follow-ups from [dates], and they merged [notable work] yesterday. Want me to write this up?

## Rules

- **Do NOT create new `- [ ]` checkboxes** in prep notes. People files typically have a DataviewJS query that aggregates open tasks. New checkboxes create duplicates. Use plain bullets instead.
- **Use bold for topic headings**, not markdown heading levels (h1-h3 are reserved for file structure).
- **Wikilink to project files** when referencing projects: `[[Projects/Project Name]]`
- **Wikilink to people** when referencing other team members: `[[People/Name]]`
- **Include GitHub PR/issue links inline** for quick reference during the meeting.
- **Idempotent**: If today's date entry already exists in the People file, update it rather than creating a duplicate.
- **This is coaching prep, not status tracking.** Frame everything through a development lens.
