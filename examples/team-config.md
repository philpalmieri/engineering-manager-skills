# Team Configuration

The team roster is a JSON file that describes your team members, collaborators, and organizational context. Skills use this to match names, determine relationship types, and fetch GitHub activity.

## Schema

```json
{
  "org": "your-github-org",
  "project": "#1234",
  "fiscal_calendar": {
    "Q1": ["Jul", "Aug", "Sep"],
    "Q2": ["Oct", "Nov", "Dec"],
    "Q3": ["Jan", "Feb", "Mar"],
    "Q4": ["Apr", "May", "Jun"]
  },
  "members": [
    {
      "name": "Jane Doe",
      "login": "janedoe",
      "vault_name": "janedoe",
      "relationship": "direct",
      "level": "IC2",
      "pronouns": "she/her",
      "focus_areas": ["backend", "API design"]
    }
  ],
  "collaborators": [
    {
      "name": "John Smith",
      "login": "johnsmith",
      "vault_name": "johnsmith",
      "relationship": "peer",
      "team": "Platform Engineering"
    },
    {
      "name": "Alice Wong",
      "login": "alicew",
      "vault_name": "alicew",
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
| `project` | No | GitHub project board number |
| `fiscal_calendar` | No | Maps fiscal quarters to calendar months |

### Members (people you manage)

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name |
| `login` | Yes | GitHub username |
| `vault_name` | Yes | Filename in `People/` directory (without `.md`) |
| `relationship` | Yes | Must be `"direct"` |
| `level` | No | IC level, grade, or equivalent |
| `pronouns` | No | Used in generated text |
| `focus_areas` | No | Technical areas for context |

### Collaborators (peers, manager, cross-functional)

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Display name |
| `login` | Yes | GitHub username |
| `vault_name` | Yes | Filename in `People/` directory |
| `relationship` | Yes | `"peer"`, `"manager"`, or `"skip-level"` |
| `team` | No | Their team name for context |

## Notes

- `vault_name` must match the People file exactly (case-sensitive)
- `login` is used for GitHub API queries (`author:{login}`, `reviewed-by:{login}`)
- The `fiscal_calendar` is optional; if omitted, skills use standard calendar quarters
- You can add any additional fields you want; skills will ignore what they don't recognize
