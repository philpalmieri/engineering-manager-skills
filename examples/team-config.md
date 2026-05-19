# Team Configuration

The team roster is a JSON file that describes your team members, collaborators, and organizational context. Skills use this to match names, determine relationship types, fetch GitHub activity, and configure reporting.

## Schema

```json
{
  "org": "your-github-org",
  "project_number": 1234,
  "team_repo": "your-org/your-team-repo",
  "discussion_sources": ["your-org/engineering"],
  "snippet": {
    "output_dir": "Projects/Snippets/",
    "archive_dir": "Archive/Projects/Snippets/",
    "noise_labels": ["oncall", "automated"],
    "incident_labels": ["incident", "sev1"],
    "key_projects": [
      { "name": "Project Alpha", "keywords": ["alpha", "new-feature"], "issue": 42 },
      { "name": "Infrastructure", "keywords": ["terraform", "k8s", "migration"] }
    ]
  },
  "vault_path": "~/Documents/Obsidian/workvault",
  "fiscal_year": {
    "q1": ["Jan", "Feb", "Mar"],
    "q2": ["Apr", "May", "Jun"],
    "q3": ["Jul", "Aug", "Sep"],
    "q4": ["Oct", "Nov", "Dec"]
  },
  "members": [
    {
      "name": "Jane Doe",
      "login": "janedoe",
      "vault_name": "Jane",
      "relationship": "direct",
      "level": "Senior",
      "pronouns": "she/her",
      "joined": "2023-06"
    }
  ],
  "collaborators": [
    {
      "name": "John Smith",
      "login": "johnsmith",
      "vault_name": "John",
      "relationship": "peer",
      "role": "Product Manager"
    },
    {
      "name": "Alice Wong",
      "vault_name": "Alice",
      "relationship": "manager"
    }
  ]
}
```

## Field reference

### Top-level

| Field | Required | Description |
|-------|----------|-------------|
| `org` | Yes | GitHub organization name for API queries |
| `project_number` | No | GitHub Project board number (for sprint tracking) |
| `team_repo` | No | Primary team repository (org/repo format) |
| `discussion_sources` | No | Repos with discussions to scan (e.g., engineering announcements) |
| `vault_path` | No | Absolute path to Obsidian vault (skills ask at runtime if missing) |
| `fiscal_year` | No | Maps fiscal quarters to calendar months (defaults to calendar year) |

### Snippet config

| Field | Required | Description |
|-------|----------|-------------|
| `snippet.output_dir` | No | Vault path for weekly snippets (default: `Projects/Snippets/`) |
| `snippet.archive_dir` | No | Vault path for archived snippets |
| `snippet.noise_labels` | No | Issue labels to exclude from reports |
| `snippet.incident_labels` | No | Issue labels that indicate incidents |
| `snippet.key_projects` | No | Array of projects to track (name, keywords, optional issue number) |

### Members (people you manage)

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name |
| `login` | Yes | GitHub username |
| `vault_name` | Yes | Filename in `People/` directory (without `.md`) |
| `relationship` | Yes | Must be `"direct"` |
| `level` | No | IC level, grade, or equivalent |
| `pronouns` | No | Used in generated text |
| `joined` | No | When they joined (YYYY-MM) |

### Collaborators (peers, manager, cross-functional)

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name |
| `login` | No | GitHub username (if they're on GitHub) |
| `vault_name` | Yes | Filename in `People/` directory |
| `relationship` | Yes | `"peer"`, `"manager"`, or `"skip-level"` |
| `role` | No | Their role for context |

## Notes

- `vault_name` must match the People file exactly (case-sensitive)
- `login` is used for GitHub API queries (`author:{login}`, `reviewed-by:{login}`)
- The `fiscal_year` is optional; if omitted, skills use standard calendar quarters
- `key_projects` keywords are matched against PR titles, branch names, and issue labels
- You can add any additional fields; skills ignore what they don't recognize
- **Keep `team.json` gitignored** — it contains real names and logins
