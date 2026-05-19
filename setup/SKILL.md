---
name: setup
description: |-
  Interactive setup wizard for Engineering Manager Skills. Walks the user through creating
  their team.json configuration file with team roster, org settings, vault path,
  and project tracking config. Run this first after installing the skills.
  Use when asked to 'set up', 'configure', or 'initialize' the manager skills.
license: MIT
---

# Setup Wizard

Walk the user through creating their `team.json` configuration file step by step. This file is the central config that all other skills reference.

## When to Trigger

- User says "setup", "configure", "initialize", or "get started"
- User just installed the skills and has no `team.json` yet
- User wants to update their team configuration

## Process

### Step 1: Locate or Create team.json

Check if `team.json` exists in the current working directory or a `data/` subdirectory.

- If it exists, ask if they want to update it or start fresh
- If it doesn't exist, create it at `./team.json` (or `./data/team.json` if a `data/` dir exists)

### Step 2: Organization and Project Config

Ask the user for:

1. **GitHub org name** (required) — e.g., "acme-corp"
2. **Team repo** (optional) — the primary repo the team works in, e.g., "acme-corp/platform"
3. **GitHub Project number** (optional) — for sprint tracking
4. **Discussion sources** (optional) — repos with discussions to monitor, e.g., ["acme-corp/engineering"]

### Step 3: Vault Configuration

Ask the user:

1. **Obsidian vault path** — full path to their vault (e.g., `~/Documents/Obsidian/workvault`)
2. **Vault directory conventions** — confirm defaults or customize:

| Directory | Purpose | Default |
|-----------|---------|---------|
| Dailies | Daily notes | `Dailies/` |
| People | People/1:1 files | `People/` |
| Projects | Project files | `Projects/` |
| Archive | Archived content | `Archive/` |
| Areas | Areas of responsibility | `Areas/` |

3. **Snippet directories:**
   - Output: where weekly snippets go (default: `Projects/Snippets/`)
   - Archive: where old snippets move to (default: `Archive/Projects/Snippets/`)

If the user doesn't use Obsidian, these can be omitted (skills that need vault access will ask at runtime).

### Step 4: Fiscal Year

Ask:
- Does your org use a non-standard fiscal year?

Default to standard calendar year:
```json
"fiscal_year": { "q1": ["Jan","Feb","Mar"], "q2": ["Apr","May","Jun"], "q3": ["Jul","Aug","Sep"], "q4": ["Oct","Nov","Dec"] }
```

### Step 5: Team Members (Direct Reports)

For each direct report, collect:

| Field | Required | Description |
|-------|----------|-------------|
| `login` | Yes | GitHub username |
| `name` | Yes | Display name |
| `pronouns` | No | e.g., "she/her" |
| `joined` | No | When they joined the team (YYYY-MM) |
| `level` | No | Job level/title (e.g., "Senior", "L9", "IC3") |
| `relationship` | Auto | Set to "direct" |
| `vault_name` | No | Filename in People/ directory (defaults to login) |

Accept bulk input (paste a list) or one-at-a-time.

### Step 6: Collaborators

For peers, manager, skip-levels:

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name |
| `login` | No | GitHub username |
| `relationship` | Yes | One of: "manager", "peer", "skip-level" |
| `vault_name` | No | Filename in People/ directory |
| `role` | No | Their role for context (e.g., "Product Manager") |

### Step 7: Key Projects (for Snippet Tracking)

Ask: "What are your team's active projects? For each, give me a name and keywords I can use to match PRs/issues to that project."

Build the `snippet.key_projects` array:
```json
{ "name": "Project Name", "keywords": ["keyword1", "keyword2"], "issue": 123 }
```

The `issue` field is optional (for linking to a tracking epic).

### Step 8: Noise/Incident Labels

Ask:
- Labels to EXCLUDE from reports (noise_labels): e.g., "oncall", "automated"
- Labels that indicate incidents (incident_labels): e.g., "incident", "sev1"

### Step 9: Schedule Conventions

How meetings appear in daily notes:

- Time-blocked format: `HH:MM - HH:MM Description` (default)
- 1:1s identified by: "1:1" prefix, or a `[[People/name]]` wikilink
- Skip non-1:1 meetings for prep (standup, training, interviews, etc.)

Ask: "How do you format your schedule in daily notes?"

### Step 10: Write and Confirm

1. Assemble the full JSON
2. Show a preview to the user
3. Write `team.json` to the chosen location
4. Remind them to add `team.json` to `.gitignore` if the repo is public

### Post-Setup

After writing, confirm:
```
✅ team.json created at {path}

Try these to start:
  "prep my day"                — full daily orchestration
  "prep 1:1 for {name}"       — 1:1 talking points
  "weekly snippet"             — draft your status update
  "analyze team last week"     — team activity report
```

## Updating

If `team.json` already exists, offer:
- Add/remove a team member
- Update org settings
- Add/update key projects
- Full reconfigure

## Validation

After writing, validate:
- All members have `login` and `name`
- All collaborators have `name` and `relationship`
- `relationship` values are one of: "direct", "manager", "peer", "skip-level"
- No duplicate logins
- `fiscal_year` has all 4 quarters covering all 12 months

## Reference: Full team.json Schema

See `team.json.example` in the repository root for a complete example with all fields.
