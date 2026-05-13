# Example Vault Structure

This is the expected Obsidian vault layout for the EM skills to work. Directory names are configurable via the setup skill, but this is the default.

```
vault/
  Dailies/
    2026-05-13.md              # Today's daily note
    2026-05-12.md              # Yesterday
    ...
  
  Archive/
    Dailies/                   # Old dailies moved here after review
      2026-04-01.md
      ...
  
  People/
    janedoe.md                 # One file per person (named by login, first name, etc.)
    johnsmith.md
    ...
  
  Projects/
    Project Alpha.md           # One file per project
    Project Beta.md
    Snippets/                  # Weekly status updates
      2026-05-12.md
    Interviews/                # Interview prep (optional)
      Scripts/
        Behavioral Grade 9.md
  
  Areas/
    Team Metrics/              # Recurring analysis (on-call reviews, etc.)
    Package Security Office Hours.md
    ...
```

## Daily note format

Daily notes use a time-blocked schedule followed by tasks:

```markdown
10:00 - 11:00 Team Standup
12:00 - 13:00 [[Areas/Office Hours]]
13:00 - 13:30 1:1 [[People/janedoe|janedoe]]
14:00 - 14:30 1:1 [[People/johnsmith|johnsmith]]
15:00 - 15:30 1:1 [[People/alicew|alicew]]

- [ ] #today Review Q3 OKRs
- [ ] #followup Check on deployment issue

### 📥 Moved Open Tasks
- [ ] Finish budget proposal ➡️➡️ #today [[Dailies/2026-05-10|2026-05-10]]
- [ ] Follow up with platform team #followup [[Dailies/2026-05-11|2026-05-11]]

## Today's Priority Focus

(DataviewJS query block here)
```

## People file format

Each person has a running notes file:

```markdown
## Open Tasks

(DataviewJS query block that aggregates open tasks referencing this person)

## Notes

#### [[Dailies/2026-05-13|2026-05-13]] 1:1

**Project Alpha** (Recognition)
- Shipped the API refactor, clean test coverage
- Call out: good initiative on the migration path

**Sprint Planning** (Follow-up)
- Check in: how's the new sprint cadence working?
- The backlog grooming item from 5/6 is still open

#### [[Dailies/2026-05-06|2026-05-06]] 1:1

**Onboarding** (Coaching)
- Discussed pairing strategy for first month
```

## Key conventions

- **h4 date headers** (`####`) make sections collapsible in Obsidian
- **Bold topic headings** with parenthetical context types
- **Plain bullets** for talking points (not `- [ ]` checkboxes, to avoid duplicate tasks)
- **Wikilinks** to People and Projects files for bidirectional linking
- **GitHub links** inline for quick reference during meetings
