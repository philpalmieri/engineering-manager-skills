# Orchestration Skills

End-to-end workflows that chain multiple skills together into complete routines.

## What These Do

Orchestration skills are the "one command to rule them all" layer. Instead of invoking 5-6 individual skills manually, you say something like "prep my day" and the orchestrator handles the full sequence: reading your schedule, prepping each 1:1, rolling tasks, scanning discussions, and setting priorities.

## Skills

| Skill | What it does |
|-------|-------------|
| **prep-my-day** | Full morning routine: reads today's schedule, rolls tasks forward, preps every 1:1 (direct reports + peers), scans engineering discussions, and sets today's priorities. |

## Prerequisites

- **Today's daily note** must already exist with a time-blocked schedule
- **`team.json`** with members and collaborators (to identify 1:1 types)
- **People files** in the vault for each person on the schedule
- **`gh` CLI** authenticated (for GitHub activity fetching during 1:1 prep)
- All sub-skills installed (task-rollover, direct-report-prep, peer-prep, today-priorities)

## How prep-my-day Works

```
prep-my-day
  │
  ├── 1. Read today's daily note schedule
  │      Parse time blocks, identify 1:1 meetings
  │      Match people to team.json (direct vs peer vs unknown)
  │
  ├── 2. Task rollover
  │      Move open tasks from previous dailies to today
  │      Mark sources as forwarded, track #today arrows
  │
  ├── 3. For each 1:1 (chronological order):
  │      ├── Direct report → direct-report-prep
  │      │    (GitHub activity + coaching + follow-ups)
  │      └── Peer/manager → peer-prep
  │           (alignment + commitments + partnership)
  │
  ├── 4. Engineering discussion scan
  │      Check configured discussion repos for relevant posts
  │      Add reading tasks to today's daily
  │
  └── 5. Today's priorities
         Scan vault for high-priority items
         Apply #today tags, manage rollover arrows
         Write DataviewJS query block to daily note
```

## Typical Usage

```
"prep my day"                    → full orchestration, stops between each person for review
"prep today, no stops"           → fast path: does everything, shows summary at end
"just prep my 1:1s"              → skips task rollover and priorities, only does meeting prep
```

## Interaction Modes

- **Default (with stops)** — After prepping each person, pauses to ask "anything to add?" before moving to the next
- **Fast path** — When you say "just do it all" or "no stops", writes everything and presents a single summary
- **Skip logic** — Automatically skips non-1:1 meetings (team standups, trainings, interviews)

## Adding New Orchestration Skills

If you want to create a new orchestration skill:

1. Create `skills/orchestration/{name}/SKILL.md`
2. Define the sequence of sub-skills it calls
3. Document which skills are prerequisites
4. Decide on interaction model (stops vs fast path)

Potential future orchestration skills:
- `end-of-week` — snippet prep + task cleanup + priority reset
- `quarterly-review` — team-health-report + review-cards + 9-box + talking points
- `new-hire-setup` — create person file + add to team.json + set up tracking

## Assumptions

- Your daily note exists before you run prep-my-day (it doesn't create the file)
- The schedule in the daily note uses time blocks that can be parsed for person names
- Each person on the schedule has a corresponding entry in `team.json`
- Sub-skills are all installed and accessible
