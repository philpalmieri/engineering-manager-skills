# Obsidian Skills

Atomic operations for managing an Obsidian vault as an engineering manager's daily operating system.

## What These Do

These are the low-level building blocks that interact directly with your Obsidian vault. They handle individual operations like moving tasks between dailies, managing people files, and searching vault content. Higher-level skills (management, orchestration) compose these.

## Skills

| Skill | What it does |
|-------|-------------|
| **task-rollover** | Moves open tasks (`- [ ]`) from previous dailies to today's note. Marks source tasks as forwarded (`[>]`), preserves origin dates, and manages `#today` rollover arrows. |
| **daily-note** | *(Placeholder)* Create and structure today's daily note with schedule and sections. |
| **people-file** | *(Placeholder)* Manage Person files: create new entries, update sections, query open items. |
| **vault-search** | *(Placeholder)* Search across vault for tagged items, open tasks, or content matching patterns. |

> Skills marked *(Placeholder)* have empty SKILL.md files and are planned for future development.

## Prerequisites

- **Obsidian vault** at the path configured in `team.json` (field: `vault_path`)
- **Daily notes** in `{vault_path}/Dailies/YYYY-MM-DD.md` format
- **People files** in `{vault_path}/People/{name}.md`
- File system access to read and write vault files

## Vault Structure Expected

```
{vault_path}/
  Dailies/
    2026-05-19.md          # today's daily note
    2026-05-18.md          # yesterday's
  Archive/
    Dailies/               # older dailies moved here after review
  People/
    Sam.md                 # person files (one per team member)
    Alex.md
  Projects/
    Some Project.md        # project reference files
  Areas/
    Team Metrics/          # analysis outputs
```

## Typical Workflow

```
Morning (usually called by prep-my-day):
1. "roll my tasks forward"       → task-rollover moves open items to today

Standalone:
2. "what's open for Sam?"        → vault-search finds open tasks mentioning Sam
3. "create daily note"           → daily-note scaffolds today's file
```

## Key Design Decisions

- **Move, don't copy** — Tasks are moved (source marked `[>]`), not duplicated across dailies
- **Origin tracking** — Each moved task retains a `[[Dailies/YYYY-MM-DD|YYYY-MM-DD]]` wikilink showing where it originated
- **#today system** — Priority items get `#today` tags with `➡️` arrows counting consecutive rollovers
- **Idempotent** — Running task-rollover twice won't duplicate tasks; already-forwarded items are skipped

## Assumptions

- Your vault uses `Dailies/YYYY-MM-DD.md` naming (not weekly notes or other formats)
- Archived dailies live at `Archive/Dailies/` (the user moves them manually after review)
- Tasks use standard Obsidian checkbox syntax: `- [ ]` (open), `- [x]` (done), `- [>]` (forwarded)
- People file names match the `vault_name` field in `team.json`
