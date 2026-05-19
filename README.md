# Engineering Manager Skills

A collection of reusable [Copilot CLI skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) for engineering managers who use Obsidian as their PKM system and GitHub for team management.

These skills are composable, generic, and contain no personal or organizational data. They assume you bring your own vault, your own team roster, and your own GitHub access.

## Installation

### Option 1: Ask Copilot to install it

Already running Copilot CLI? Just ask:

> install skills from philpalmieri/engineering-manager-skills

Copilot will clone the repo and symlink all skills into `~/.copilot/skills/` for you.

### Option 2: Clone and symlink

```bash
gh repo clone philpalmieri/engineering-manager-skills ~/Dev/engineering-manager-skills

# Symlink all skills into your Copilot skills directory
for skill in ~/Dev/engineering-manager-skills/skills/*/*; do
  ln -sf "$skill" ~/.copilot/skills/$(basename "$skill")
done
```

### Option 3: Install individual skills

```bash
# Clone the repo, then copy just the skills you want
cp -r skills/management/direct-report-prep ~/.copilot/skills/direct-report-prep
cp -r skills/orchestration/prep-my-day ~/.copilot/skills/prep-my-day
```

### Option 4: Use as a repo-level skillset

The repo includes a `.copilot/skills` symlink that points to `../skills`. If you clone this repo into your workspace, Copilot CLI will load the skills automatically when you work in that directory.

```bash
gh repo clone philpalmieri/engineering-manager-skills
cd engineering-manager-skills
# Skills are available via .copilot/skills symlink
```

### First run

After installing, run the **setup** skill to create your `team.json`:
```
> setup my team config
```

This walks you through creating the config file all other skills depend on. See `team.json.example` for the full schema.

## What's in here

### Obsidian Skills (`skills/obsidian/`)
Atomic operations for managing an Obsidian vault as an EM's daily operating system.

| Skill | Description |
|-------|-------------|
| `task-rollover` | Move open tasks from previous dailies to today's note (mark source as forwarded) |

### GitHub Skills (`skills/github/`)
Team data fetching and analysis via the `gh` CLI.

| Skill | Description |
|-------|-------------|
| `team-activity` | Fetch PRs, reviews, and issues for team members |
| `sprint-status` | Query project boards for sprint progress and health |
| `sprint-management` | Automate sprint lifecycle (start, end, rollover) |
| `oncall-review` | Assess on-call shift quality from issue engagement |
| `epic-cleanup` | Audit active epics for ownership gaps and closure candidates |
| `epic-overview` | Grouped overview of all active epics by strategic objective |

### Management Skills (`skills/management/`)
Higher-level workflows that compose Obsidian + GitHub skills.

| Skill | Description |
|-------|-------------|
| `direct-report-prep` | 1:1 prep for direct reports (activity + coaching + follow-ups) |
| `peer-prep` | 1:1 prep for peers and cross-functional partners |
| `weekly-snippet` | Generate weekly status updates from vault + GitHub + sprint data |
| `team-analysis` | Team activity reports over configurable date ranges |
| `today-priorities` | Scan vault for high-priority items, tag with #today system |
| `project-status` | Weekly project status report from actual GitHub signals |
| `interview-prep` | Generate resume-specific behavioral interview scripts |
| `interview-analysis` | Clean up raw interview notes into structured summaries |

### Technical Skills (`skills/technical/`)
Research and documentation generation.

| Skill | Description |
|-------|-------------|
| `product-brief` | Generate a product summary for stakeholders |
| `technical-deep-dive` | Codebase analysis and architecture review |
| `initiative-breakdown` | Deep-dive breakdown of a GitHub initiative or epic |

### Metrics Skills (`skills/metrics/`)
Data-driven team health analysis and visualization.

| Skill | Description |
|-------|-------------|
| `review-network` | Who reviews whom: network density, reciprocity, silo detection |
| `review-cycles` | PR feedback rounds before merge, per person and trending |
| `project-people-map` | Project ↔ people heatmap, bus factor, silo alerts |
| `contribution-velocity` | Time-to-merge, throughput, review responsiveness |
| `review-cards` | Quarterly grade cards per person (level-adjusted) |
| `nine-box` | 9-box talent grid (performance vs growth trajectory) |
| `team-health-report` | Orchestration: runs all metrics, produces unified dashboard |

### Orchestration Skills (`skills/orchestration/`)
Chain multiple skills into end-to-end workflows.

| Skill | Description |
|-------|-------------|
| `prep-my-day` | Full day prep: task rollover, 1:1 prep, discussion scan, priorities |

## Configuration

All skills reference a `team.json` file for org-specific config. This keeps skills generic and publishable while your actual team data stays private.

### Quick start

```bash
cp team.json.example team.json
# Edit with your actual team data
```

### Key config sections

| Section | Purpose |
|---------|---------|
| `org` | GitHub organization for API queries |
| `team_repo` | Primary team repo |
| `project_number` | GitHub Project board for sprint tracking |
| `discussion_sources` | Repos with engineering discussions to monitor |
| `snippet` | Weekly snippet config: output dirs, key projects, noise labels |
| `fiscal_year` | Quarter-to-month mapping |
| `members` | Direct reports with GitHub logins and vault file names |
| `collaborators` | Peers, manager, skip-levels |

See `team.json.example` for the complete schema with all fields documented.

## Prerequisites

- [GitHub Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) installed
- [GitHub CLI (`gh`)](https://cli.github.com/) authenticated with access to your org
- An Obsidian vault (any structure; see `examples/vault-structure.md` for conventions)
- A `team.json` config file (run the setup skill or copy the example)

## Composability

Skills are designed to be chained. The orchestration skills call atomic skills in sequence:

```
prep-my-day
  ├── task-rollover        (move yesterday's open tasks, mark as forwarded)
  ├── direct-report-prep   (for each direct 1:1 on the schedule)
  ├── peer-prep            (for each peer 1:1 on the schedule)
  ├── discussion-scan      (check engineering discussions for relevant posts)
  └── today-priorities     (scan and tag priorities with #today)
```

You can also invoke any skill independently.

## Privacy

- `team.json` is gitignored by default (contains real names/logins)
- Skills contain zero personal or organizational data
- All org-specific config lives exclusively in `team.json`

## Contributing

Skills should be:
- **Generic** — no personal data, org names, or hardcoded paths
- **Atomic** — do one thing well (orchestration skills can compose them)
- **Documented** — clear SKILL.md with triggers, process, and output format
- **Config-driven** — reference `team.json` for anything org-specific

## License

MIT
