---
name: task-rollover
description: |-
  Move open tasks from previous daily notes to today's daily note in an Obsidian vault.
  Handles rollover arrows for recurring items, source date tracking, and cleanup of
  source files. Use when prepping today's daily note or when asked to roll tasks forward.
---

# Task Rollover

Move all open tasks (`- [ ]`) from previous daily notes into today's daily note, then remove them from the source. This prevents duplicate tasks across multiple dailies.

## When to use

- During daily prep ("prep my day", "roll tasks forward", "move yesterday's tasks")
- Any time you notice open tasks sitting on old daily notes

## Prerequisites

- Obsidian vault with daily notes in a `Dailies/` directory
- Daily note filenames follow `YYYY-MM-DD.md` format
- Today's daily note should already exist (create it first if not)

## Process

### Step 1: Scan for open tasks

Search all files in `Dailies/` with dates BEFORE today for unchecked tasks (`- [ ]`).

```bash
grep -rl '^\- \[ \]' <vault_path>/Dailies/ | sort
```

Exclude today's daily from the results.

### Step 2: Collect open tasks from each source daily

For each daily note with open tasks:

1. **Standalone tasks**: Lines starting with `- [ ]` that are NOT indented inside meeting notes or other structured content. These are top-level tasks below a `---` separator or in a "Moved Open Tasks" section.

2. **Nested tasks in meeting notes**: Tasks (`- [ ]`) indented inside meeting records. These need special handling (see Step 5).

### Step 3: Process each task for today's note

For each standalone open task:

1. **Add a rollover arrow** (`➡️`) if the task has a `#today` tag. Place the arrow before `#today`. If it already has arrows, add one more.
   - Tasks WITHOUT `#today` get moved as-is (no arrow added).
   - The arrow count represents how many days the task has rolled over while tagged `#today`.

2. **Add a source date suffix** if the task doesn't already have one. Format: `[[Dailies/YYYY-MM-DD|YYYY-MM-DD]]` using the source daily's date. If the task already has a date suffix (from a previous rollover), keep the ORIGINAL source date, not the intermediate daily.

3. **Preserve everything else** on the task line: tags, links, emoji, formatting.

### Step 4: Write to today's daily

Add a `### 📥 Moved Open Tasks` section to today's daily note (before the Priority Focus section if one exists).

Sort tasks by rollover arrow count (most arrows first). This surfaces the stickiest items at the top.

**Rollover escalation thresholds:**
- 2+ arrows: mention these items need attention when presenting to the user
- 3+ arrows: flag with ⚠️ and suggest the user make a decision
- 4+ arrows: flag with 🚨 and be direct that the item needs to either happen today or be removed from `#today`

### Step 5: Clean up source dailies

For each source daily note:

1. **Remove standalone open tasks** entirely (they now live on today's daily).
2. **Convert nested meeting-note tasks** from `- [ ]` to plain `- ` (remove the checkbox). This preserves the meeting context while preventing duplicate task appearances in Obsidian's task queries.
3. **Keep completed tasks** (`- [x]`) in place. They're part of the historical record.
4. **Keep the Priority Focus section** and any DataviewJS blocks. They're part of that day's record.

### Step 6: Verify

After all moves:
- Confirm zero `- [ ]` remain in any daily note before today
- Confirm today's daily has all tasks accounted for
- Report summary: "Moved N tasks from M daily notes. X items flagged for escalation."

## Rules

- **MOVE, not copy.** Tasks must be removed from source after adding to target. No duplicates.
- **One arrow per rollover.** Only add ➡️ when moving a `#today` task forward. Do not add arrows in-place without moving.
- **Preserve original source dates.** The `[[Dailies/...]]` suffix should always point to where the task was FIRST created, not the last daily it sat on.
- **Never modify completed tasks.** `- [x]` items stay where they are.
- **Never move tasks from today's own daily.** Only process dailies with dates strictly before today.
- **Idempotent.** Running this twice on the same day should not create duplicates or add extra arrows.

## Example

**Before (2026-05-12.md):**
```markdown
- [ ] ➡️ #today post update about ticket cleanup
- [ ] #followup Send summary to Rohan
- [ ] Check on issue 4661 #followup [[Dailies/2026-04-20|2026-04-20]]
```

**After moving to 2026-05-13.md:**
```markdown
### 📥 Moved Open Tasks
- [ ] ➡️➡️ #today post update about ticket cleanup [[Dailies/2026-05-12|2026-05-12]]
- [ ] #followup Send summary to Rohan [[Dailies/2026-05-12|2026-05-12]]
- [ ] Check on issue 4661 #followup [[Dailies/2026-04-20|2026-04-20]]
```

**2026-05-12.md after cleanup:**
Open tasks removed. Completed tasks and meeting notes preserved.
