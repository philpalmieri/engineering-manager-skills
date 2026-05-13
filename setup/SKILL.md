---
name: setup
description: |-
  Configure the Copilot EM Skills environment. Set vault path, team roster location,
  and vault conventions. Run this first before using other skills.
---

# Setup

Configure the environment for Copilot EM Skills. This skill collects the information other skills need to operate.

## When to use

- First time using these skills
- When you change vault location, team roster, or conventions

## What it configures

### 1. Vault path

The absolute path to your Obsidian vault.

**Ask the user:**
> What is the full path to your Obsidian vault?

Example: `~/Documents/Obsidian/workvault`

### 2. Vault directory conventions

Confirm or customize directory names. Defaults:

| Directory | Purpose | Default |
|-----------|---------|---------|
| Dailies | Daily notes | `Dailies/` |
| People | People/1:1 files | `People/` |
| Projects | Project files | `Projects/` |
| Archive | Archived content | `Archive/` |
| Areas | Areas of responsibility | `Areas/` |

**Ask the user:**
> Do you use these default directory names, or do you have custom names?

### 3. Daily note format

- Filename format: `YYYY-MM-DD.md` (default)
- Location: `<vault>/Dailies/`
- Archive location for old dailies: `<vault>/Archive/Dailies/`

**Ask the user:**
> Do your daily notes use YYYY-MM-DD.md format? Are archived dailies in Archive/Dailies/?

### 4. People file conventions

How People files are structured:

- **Filename**: matches a unique identifier (GitHub login, first name, etc.)
- **Date entries**: `#### [[Dailies/YYYY-MM-DD|YYYY-MM-DD]]` as h4 headers
- **Topic format**: `**Topic** (Context Type)` with plain bullet talking points
- **Task handling**: DataviewJS query block aggregates open tasks; avoid creating duplicate checkboxes in prep notes

**Ask the user:**
> How are your People files named? By first name, GitHub login, or something else?

### 5. Team roster

A JSON file containing team member information. Expected schema:

```json
{
  "org": "your-github-org",
  "members": [
    {
      "name": "Display Name",
      "login": "github-login",
      "vault_name": "PeopleFileName",
      "relationship": "direct",
      "level": "IC2",
      "pronouns": "they/them"
    }
  ],
  "collaborators": [
    {
      "name": "Peer Name",
      "login": "peer-login",
      "vault_name": "PeerFileName",
      "relationship": "peer"
    }
  ]
}
```

**Ask the user:**
> Where is your team roster file? (Or should I help you create one?)

### 6. Schedule conventions

How meetings appear in daily notes:

- Time-blocked format: `HH:MM - HH:MM Description`
- 1:1s identified by: "1:1" prefix, or a `[[People/name]]` wikilink
- Skip non-1:1 meetings for prep (standup, training, interviews, etc.)

**Ask the user:**
> How do you format your schedule in daily notes? Do you pre-populate it or generate from a calendar?

## Output

After collecting answers, store the configuration as memories and confirm:

```
✅ Vault: ~/Documents/Obsidian/workvault
✅ Dailies: Dailies/ (archive: Archive/Dailies/)
✅ People: People/ (named by GitHub login)
✅ Team roster: data/team.json (4 directs, 3 collaborators)
✅ Schedule format: time-blocked with wikilinks
```

Skills will use these conventions automatically.
