# Engineering Manager Skills

A collection of reusable [Copilot CLI skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) for engineering managers who use Obsidian as their PKM system and GitHub for team management.

These skills are composable, generic, and contain no personal or organizational data. They assume you bring your own vault, your own team roster, and your own GitHub access.

## How It Works

You talk to Copilot like you'd talk to a chief of staff:

```
> sync my team data
> prep my day
> how's the team doing?
> prep 1:1 with Sam
> show me the review network for last quarter
> write my weekly snippet
> who's siloed on what projects?
> generate review cards for Q3
```

Behind each prompt is a skill that knows where to look, what to fetch, and how to format the output. Skills pull from GitHub (PRs, reviews, issues, sprints), your Obsidian vault (daily notes, people files, open tasks), and a local `team.json` config. No new apps, no dashboards to check, no manual data entry. Just conversation.

> [!IMPORTANT]
> **This is signal, not the whole picture.**
>
> Everything here automates data collection you'd otherwise do manually: PR counts, review patterns, sprint progress, open tasks. It's useful. But it's one signal among many.
>
> Code activity doesn't capture mentoring conversations, design work, cross-team coordination, oncall heroics, or the context behind why someone's PR count dropped. These tools aggregate raw data. They have no concept of nuance, personal circumstances, or what's happening outside of GitHub.
>
> Use this to *inform* your judgment, not replace it. Pair it with the things that actually make you a good manager: paying attention, asking questions, and knowing your people.
>
> And because it's AI doing the analysis: **trust but verify. Always.**

## What's Included

28 skills across 6 categories. Each category has its own README with detailed documentation, prerequisites, and usage examples.

### [Obsidian Skills](skills/obsidian/) (`skills/obsidian/`)
Atomic vault operations: task movement, daily notes, people file management.

| Skill | Description |
|-------|-------------|
| [`task-rollover`](skills/obsidian/task-rollover/SKILL.md) | Move open tasks from previous dailies to today (mark source as forwarded) |
| `daily-note` | *(Planned)* Create and structure today's daily note |
| `people-file` | *(Planned)* Manage Person files and entries |
| `vault-search` | *(Planned)* Search vault for tagged items and open tasks |

### [GitHub Skills](skills/github/) (`skills/github/`)
Data pipeline: fetch, enrich, and sync team activity from GitHub.

| Skill | Description |
|-------|-------------|
| [`team-activity`](skills/github/team-activity/SKILL.md) | Fetch PRs, reviews, and issues for team members |
| [`enrich-prs`](skills/github/enrich-prs/SKILL.md) | Fetch per-PR review detail (rounds, dismissals, reviewer actions) |
| [`team-data-sync`](skills/github/team-data-sync/SKILL.md) | Smart sync: detect stale/missing data, fetch delta, enrich new PRs |
| [`sprint-status`](skills/github/sprint-status/SKILL.md) | Query project boards for sprint progress and health |
| [`sprint-management`](skills/github/sprint-management/SKILL.md) | Automate sprint lifecycle (start, end, rollover) |
| [`oncall-review`](skills/github/oncall-review/SKILL.md) | Assess on-call shift quality from issue engagement |
| [`epic-cleanup`](skills/github/epic-cleanup/SKILL.md) | Audit active epics for ownership gaps and closure candidates |
| [`epic-overview`](skills/github/epic-overview/SKILL.md) | Grouped overview of all active epics by strategic objective |

### [Management Skills](skills/management/) (`skills/management/`)
Day-to-day EM workflows: 1:1 prep, status reporting, hiring, prioritization.

| Skill | Description |
|-------|-------------|
| [`direct-report-prep`](skills/management/direct-report-prep/SKILL.md) | 1:1 prep for direct reports (activity + coaching + follow-ups) |
| [`peer-prep`](skills/management/peer-prep/SKILL.md) | 1:1 prep for peers and cross-functional partners |
| [`weekly-snippet`](skills/management/weekly-snippet/SKILL.md) | Generate weekly status updates from vault + GitHub + sprint data |
| [`team-analysis`](skills/management/team-analysis/SKILL.md) | Team activity reports over configurable date ranges |
| [`today-priorities`](skills/management/today-priorities/SKILL.md) | Scan vault for high-priority items, tag with #today system |
| [`project-status`](skills/management/project-status/SKILL.md) | Weekly project status report from actual GitHub signals |
| [`interview-prep`](skills/management/interview-prep/SKILL.md) | Generate resume-specific behavioral interview scripts |
| [`interview-analysis`](skills/management/interview-analysis/SKILL.md) | Clean up raw interview notes into structured summaries |

### [Technical Skills](skills/technical/) (`skills/technical/`)
Research and documentation for technical projects and initiatives.

| Skill | Description |
|-------|-------------|
| [`product-brief`](skills/technical/product-brief/SKILL.md) | Generate a product summary for stakeholders |
| [`technical-deep-dive`](skills/technical/technical-deep-dive/SKILL.md) | Codebase analysis and architecture review |
| [`initiative-breakdown`](skills/technical/initiative-breakdown/SKILL.md) | Deep-dive breakdown of a GitHub initiative or epic |

### [Metrics Skills](skills/metrics/) (`skills/metrics/`)
Data-driven team health: collaboration patterns, velocity, talent assessment.

| Skill | Description |
|-------|-------------|
| [`review-network`](skills/metrics/review-network/SKILL.md) | Who reviews whom: network density, reciprocity, silo detection |
| [`review-cycles`](skills/metrics/review-cycles/SKILL.md) | PR feedback rounds before merge, per person and trending |
| [`project-people-map`](skills/metrics/project-people-map/SKILL.md) | Project ↔ people heatmap, bus factor, silo alerts |
| [`contribution-velocity`](skills/metrics/contribution-velocity/SKILL.md) | Time-to-merge, throughput, review responsiveness |
| [`review-cards`](skills/metrics/review-cards/SKILL.md) | Quarterly grade cards per person (level-adjusted) |
| [`nine-box`](skills/metrics/nine-box/SKILL.md) | 9-box talent grid (performance vs growth trajectory) |
| [`team-health-report`](skills/metrics/team-health-report/SKILL.md) | Orchestration: runs all metrics, produces unified dashboard |

### [Orchestration Skills](skills/orchestration/) (`skills/orchestration/`)
End-to-end workflows that chain multiple skills together.

| Skill | Description |
|-------|-------------|
| [`prep-my-day`](skills/orchestration/prep-my-day/SKILL.md) | Full day prep: task rollover, 1:1 prep, discussion scan, priorities |

---

## Getting Started

### 1. Install the skills

**Easiest:** Ask Copilot CLI directly:

> install skills from philpalmieri/engineering-manager-skills

**Manual:** Clone and symlink:

```bash
gh repo clone philpalmieri/engineering-manager-skills ~/Dev/engineering-manager-skills

for skill in ~/Dev/engineering-manager-skills/skills/*/*; do
  ln -sf "$skill" ~/.copilot/skills/$(basename "$skill")
done
```

### 2. Configure your team

Run the setup skill to create your `team.json`:

```
> setup my team config
```

This walks you through adding your org, team members, vault path, and project board. The resulting `team.json` is what every other skill reads from. See `team.json.example` for the full schema.

### 3. Pull your first data

```
> sync my team data
```

This fetches the current and previous month of GitHub activity (PRs, reviews, issues) for everyone on your team and enriches PRs with review detail. Takes 1-3 minutes depending on team size.

### 4. Try it out

Here's a typical first-day flow:

```
> prep my day                     # reads today's schedule, preps all 1:1s, sets priorities
> how's the team doing?           # runs team-health-report with current data
> review network for last month   # shows who's reviewing whom
> prep my snippet                 # drafts your weekly status update
```

### 5. Build up data over time

The longer you use it, the more useful metrics become. After a month of data, you get trend lines. After a quarter, you can run review cards and 9-box assessments:

```
> sync last quarter
> generate review cards for Q3
> do 9-box for the team
```

---

## Installation Options

### Option 1: Ask Copilot

> install skills from philpalmieri/engineering-manager-skills

### Option 2: Clone and symlink

```bash
gh repo clone philpalmieri/engineering-manager-skills ~/Dev/engineering-manager-skills

for skill in ~/Dev/engineering-manager-skills/skills/*/*; do
  ln -sf "$skill" ~/.copilot/skills/$(basename "$skill")
done
```

### Option 3: Individual skills only

```bash
# Copy just what you need
cp -r skills/management/direct-report-prep ~/.copilot/skills/direct-report-prep
cp -r skills/orchestration/prep-my-day ~/.copilot/skills/prep-my-day
```

### Option 4: Repo-level loading

The repo includes a `.copilot/skills` symlink. Clone it into your workspace and skills load automatically when you `cd` into the directory.

```bash
gh repo clone philpalmieri/engineering-manager-skills
cd engineering-manager-skills
```

## Configuration

All skills reference a `team.json` file for org-specific config. This keeps skills generic and publishable while your actual team data stays private.

```bash
cp team.json.example team.json   # then edit with your real data
# — or —
> setup my team config            # interactive setup via Copilot
```

| Section | Purpose |
|---------|---------|
| `org` | GitHub organization for API queries |
| `team_repo` | Primary team repo |
| `project_number` | GitHub Project board for sprint tracking |
| `discussion_sources` | Repos with engineering discussions to monitor |
| `snippet` | Weekly snippet config: output dirs, key projects, noise labels |
| `fiscal_year` | Quarter-to-month mapping |
| `vault_path` | Absolute path to your Obsidian vault |
| `members` | Direct reports with GitHub logins and vault file names |
| `collaborators` | Peers, manager, skip-levels |

See `team.json.example` for the complete schema with all fields documented.

## Prerequisites

- [GitHub Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) installed and working
- [GitHub CLI (`gh`)](https://cli.github.com/) authenticated with access to your org
- An [Obsidian](https://obsidian.md/) vault with daily notes and people files
- A `team.json` config file (run the setup skill or copy the example)

## Architecture

Skills are layered and composable:

```
┌─────────────────────────────────────────────────────────┐
│  Orchestration       prep-my-day, team-health-report    │
├─────────────────────────────────────────────────────────┤
│  Management          1:1 prep, snippets, interviews     │
│  Technical           product briefs, deep dives         │
│  Metrics             review network, velocity, 9-box    │
├─────────────────────────────────────────────────────────┤
│  GitHub (data)       team-activity, enrich-prs, sync    │
│  Obsidian (vault)    task-rollover, vault operations    │
├─────────────────────────────────────────────────────────┤
│  Config              team.json                          │
└─────────────────────────────────────────────────────────┘
```

**Data flow:**
```
gh CLI → team-activity → data/YYYY-MM/*.json → enrich-prs → data/enrichment/*.json
                                    ↓
                    metrics skills → reports/metrics/*.md
                    management skills → vault People files
                    orchestration → chains it all together
```

## Privacy

- `team.json` is gitignored (contains real names/logins)
- `data/` is gitignored (contains fetched GitHub activity)
- Skills contain zero personal or organizational data
- All org-specific config lives exclusively in `team.json`

## Contributing

Skills should be:
- **Generic** — no personal data, org names, or hardcoded paths
- **Atomic** — do one thing well (orchestration skills can compose them)
- **Documented** — clear SKILL.md with triggers, process, and output format
- **Config-driven** — reference `team.json` for anything org-specific

Each skill category has its own README with conventions and examples.

## License

MIT
