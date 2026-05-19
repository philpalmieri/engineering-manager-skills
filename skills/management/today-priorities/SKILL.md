---
name: today-priorities
description: |-
  Scan the Obsidian vault for high-priority open items and organize them into today's
  focus list. Handles priority markers, overdue items, stale carry-overs, and the
  #today tagging system. Use when asked about today's priorities or focus.
---

# Today's Priorities

Scan the vault for actionable items and organize them into a prioritized focus list for today.

## When to use

- "Today's priorities", "what should I focus on today", "prep today tasks"
- As part of `prep-my-day` orchestration (after task rollover and 1:1 prep)

## Process

### Step 1: Read today's daily note

Check what's already scheduled (meetings, 1:1s, deadlines) and any existing todos or focus items.

### Step 2: Scan for high-priority open items

Search the vault for unchecked tasks with priority signals:

| Signal | Meaning |
|--------|---------|
| `⏫` or `🔺` | High priority marker |
| `📅 YYYY-MM-DD` | Due date (flag if today or overdue) |
| `⏳ YYYY-MM-DD` | Scheduled date (flag if today) |
| `#followup` | Needs follow-up action |
| `#figureout` | Needs investigation/decision |
| `#today` | Already tagged as today's priority |

Search locations: `People/`, `Projects/`, `Dailies/` (last 3-5 days), `Areas/`

### Step 3: Check for carry-over items

Read last 3 daily notes for open `- [ ]` items that haven't been completed. Items appearing in multiple consecutive dailies are likely important and stuck.

### Step 4: Check for time-sensitive items

- Deadlines within 48 hours
- Items tied to today's meetings (prep work, deliverables)
- Anything in People files tied to today's 1:1s

### Step 5: Present grouped by urgency

Present items to the user in three groups:
- **Must do** — deadlines, meeting prep, overdue items
- **Should do** — follow-ups, active project work
- **If time allows** — nice-to-haves, exploration items

### Step 6: Tag confirmed items

For items the user confirms as today's priorities, add `#today` to the task line **in its source file**. Tasks are never moved between files for this purpose; the tag is added in place.

### Step 7: Write Priority Focus section

Add a "Today's Priority Focus" section to today's daily note with a live query block:

```markdown
## Today's Priority Focus

\`\`\`dataviewjs
dv.taskList(
  dv.pages('"People" or "Projects" or "Dailies" or "Areas"')
    .where(p => p.file.path !== dv.current().file.path)
    .file.tasks
    .where(t => !t.completed && t.text && t.text.includes("#today"))
    .sort((a, b) =>
      ((b.text || "").match(/➡️/g) ?? []).length -
      ((a.text || "").match(/➡️/g) ?? []).length
    )
)
\`\`\`
```

This DataviewJS query surfaces all open `#today` items across the vault, sorted by rollover count.

## The #today tag system

- `#today` is added **in place** on the original task line wherever it lives
- Tasks are never duplicated or moved between files for priority tracking
- The DataviewJS query in the daily note is the live view
- When a task is completed, the user checks it off normally
- Stale `#today` tags (from previous days) are handled by the `task-rollover` skill
- Only add `➡️` arrows when the user **confirms** keeping the item for another day (not automatically)
- Place `➡️` markers before the `#today` tag for readability
- When removing `#today`, also remove all `➡️` markers

## Stale #today cleanup

If items tagged `#today` are found on previous days' dailies (not yet rolled over), present them:

> These N items still have #today from yesterday. Keep them for today, or remove the tag?

Do not remove `#today` tags without asking.
