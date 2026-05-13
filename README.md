# Copilot EM Skills

A collection of reusable [Copilot CLI skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) for engineering managers who use Obsidian as their PKM system and GitHub for team management.

These skills are composable, generic, and contain no personal or organizational data. They assume you bring your own vault, your own team roster, and your own GitHub access.

## What's in here

### Obsidian Skills
Atomic operations for managing an Obsidian vault as an EM's daily operating system.

| Skill | Description |
|-------|-------------|
| `task-rollover` | Move open tasks from previous dailies to today's note |
| `daily-note` | Create and manage daily notes with schedule and task sections |
| `vault-search` | Search for tags, tasks, and patterns across the vault |
| `people-file` | Manage People files (1:1 notes, talking points, follow-ups) |

### GitHub Skills
Team data fetching and analysis via the `gh` CLI.

| Skill | Description |
|-------|-------------|
| `team-activity` | Fetch PRs, reviews, and issues for team members |
| `sprint-status` | Query project boards for sprint progress |
| `oncall-review` | Assess on-call shift quality from issue templates |
| `issue-triage` | Bulk triage helper for open issues |

### Management Skills
Higher-level workflows that compose Obsidian + GitHub skills.

| Skill | Description |
|-------|-------------|
| `direct-report-prep` | 1:1 prep for direct reports (activity + coaching) |
| `peer-prep` | 1:1 prep for peers and cross-functional partners |
| `weekly-snippet` | Generate weekly status updates from vault + GitHub data |
| `team-analysis` | Team activity reports over configurable date ranges |
| `today-priorities` | Scan vault for high-priority items, tag and organize |

### Technical Skills
Research and documentation generation.

| Skill | Description |
|-------|-------------|
| `product-brief` | Generate a product summary for stakeholders |
| `technical-deep-dive` | Codebase analysis and architecture review |

### Orchestration Skills
Chain multiple skills into end-to-end workflows.

| Skill | Description |
|-------|-------------|
| `prep-my-day` | Full day prep: task rollover, 1:1 prep, priorities |
| `end-of-day` | End of day: task review, reflection, tomorrow prep |

## Prerequisites

- [GitHub Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) installed
- [GitHub CLI (`gh`)](https://cli.github.com/) authenticated with access to your org
- An Obsidian vault (any structure, but see `examples/vault-structure.md` for conventions)
- A team roster file (see `examples/team-config.md`)

## Setup

1. Clone this repo or install individual skills:
   ```bash
   # Clone the whole collection
   git clone https://github.com/YOUR_USER/copilot-em-skills.git

   # Or install a specific skill to your personal skills directory
   cp -r skills/obsidian/task-rollover ~/.copilot/skills/task-rollover
   ```

2. Run the setup skill to configure your environment:
   ```
   /skills → select "setup"
   ```
   This will prompt you for your vault path, team roster location, and vault conventions.

3. Skills will auto-detect based on your prompts, or invoke them explicitly.

## Configuration

Skills expect a few things from your environment:

### Team Roster (`team.json`)
A JSON file describing your team. Location is configurable via the setup skill. See `examples/team-config.md` for the schema.

### Vault Conventions
Skills work with any Obsidian vault that follows these conventions:
- Daily notes in a `Dailies/` directory (name format: `YYYY-MM-DD.md`)
- People files in a `People/` directory
- Projects in a `Projects/` directory
- Archive in an `Archive/` directory

See `examples/vault-structure.md` for a full example layout.

## Composability

Skills are designed to be chained. The orchestration skills (like `prep-my-day`) call atomic skills in sequence:

```
prep-my-day
  ├── task-rollover      (move yesterday's open tasks)
  ├── daily-note         (ensure today's note exists)
  ├── direct-report-prep (for each 1:1 on the schedule)
  ├── peer-prep          (for each peer 1:1 on the schedule)
  └── today-priorities   (scan and tag priorities)
```

You can also invoke any skill independently.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Skills should be:
- Generic (no personal data, org names, or hardcoded paths)
- Atomic (do one thing well)
- Documented (clear SKILL.md with usage examples)
- Tested against the example vault structure

## License

MIT
