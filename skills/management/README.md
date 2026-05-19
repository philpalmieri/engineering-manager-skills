# Management Skills

Day-to-day engineering management workflows: 1:1 prep, status reporting, hiring, and prioritization.

## What These Do

These are the skills you'll use most often. They combine vault context (People files, daily notes) with live GitHub activity to produce actionable prep materials, status reports, and interview scripts.

## Skills

| Skill | What it does |
|-------|-------------|
| **direct-report-prep** | Prepares coaching-oriented 1:1 talking points for a direct report. Pulls recent GitHub activity, vault follow-ups, and feedback items into structured prep. |
| **peer-prep** | Prepares partnership-oriented 1:1 talking points for peers, managers, or cross-functional collaborators. Focuses on alignment and commitments, not coaching. |
| **weekly-snippet** | Drafts your weekly status update by combining GitHub team activity, sprint data, vault notes, and tagged ideas into a structured snippet. |
| **team-analysis** | Generates a team activity report for any date range. Analyzes PR volume, review patterns, project distribution, and individual contributions. |
| **today-priorities** | Scans the vault for high-priority open items and organizes them into today's focus list with the `#today` tagging system. |
| **project-status** | Produces a weekly project status report inferred from actual GitHub signals (not manual updates). |
| **interview-prep** | Given a resume and job level template, generates a tailored behavioral interview script with resume-specific follow-ups. |
| **interview-analysis** | Transforms raw interview notes into a structured summary with rubric assessment and hire/no-hire recommendation. |

## Prerequisites

- **`team.json`** with members, collaborators, and org config
- **Obsidian vault** with People files (`People/{name}.md`) and daily notes (`Dailies/`)
- **`gh` CLI** authenticated (for GitHub activity fetching)
- **Interview templates** in your vault (for interview-prep and interview-analysis)

## Vault Conventions

These skills expect:

- People files at `{vault_path}/People/{vault_name}.md` (where `vault_name` comes from `team.json`)
- Daily notes at `{vault_path}/Dailies/YYYY-MM-DD.md`
- Archived dailies at `{vault_path}/Archive/Dailies/YYYY-MM-DD.md`
- Date entries as h4 headers: `#### [[Dailies/YYYY-MM-DD|YYYY-MM-DD]] 1:1`
- Open tasks as `- [ ]` checkboxes, plain bullets for prep notes

## Typical Workflow

```
Morning:
1. "prep 1:1 with Sam"           → direct-report-prep writes talking points
2. "prep 1:1 with Alex"          → peer-prep writes partnership topics
3. "today's priorities"           → scans vault, tags items with #today

Weekly:
4. "prep my snippet"             → weekly-snippet drafts status update
5. "project status report"       → project-status infers from GitHub signals

Hiring:
6. "prep interview for [name]"   → interview-prep builds a tailored script
7. "analyze interview notes"     → interview-analysis structures raw notes
```

## Key Design Decisions

- **Plain bullets, not tasks** — Prep notes use `-` not `- [ ]` to avoid duplicating tasks already tracked elsewhere in the vault
- **Idempotent writes** — If today's entry already exists in a Person file, it updates rather than duplicating
- **Config-driven** — All names, logins, and org references come from `team.json`
- **Vault is source of truth** — Skills read from and write to Obsidian; they don't maintain separate state

## Assumptions

- You use Obsidian as your primary note-taking/PKM system
- Each team member has a dedicated Person file in the vault
- You maintain daily notes with time-blocked schedules
- Interview templates follow a structured format with numbered questions and rubric criteria
