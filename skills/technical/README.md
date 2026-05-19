# Technical Skills

Research, documentation, and deep-dive analysis for technical projects and initiatives.

## What These Do

These skills help you understand, communicate, and plan around technical work. They pull data from GitHub (issues, PRs, code) and produce structured analysis documents suitable for engineering planning, stakeholder communication, or onboarding.

## Skills

| Skill | What it does |
|-------|-------------|
| **product-brief** | Generates a product-oriented summary: the problem, solution, complexity, user value, what's shipped, what's in progress, and what needs product attention. Good for PM syncs or leadership presentations. |
| **technical-deep-dive** | Detailed technical analysis from actual code, schemas, and architecture. Covers how things work today, risk assessment, and sprint-level sequencing. Use for architecture reviews or engineer onboarding. |
| **initiative-breakdown** | Deep-dive breakdown of a GitHub initiative or epic issue. Fetches the issue, all sub-issues, related PRs, and comments to produce a comprehensive project history and status. |

## Prerequisites

- **`gh` CLI** authenticated with access to the repos you're analyzing
- **GitHub issues/epics** that track the work you want to analyze
- **Code access** (for technical-deep-dive; it reads actual source files)

## When to Use Which

| Situation | Skill |
|-----------|-------|
| "Explain this project to a PM" | product-brief |
| "I need to onboard an engineer to this system" | technical-deep-dive |
| "Where are we on initiative #1234?" | initiative-breakdown |
| "What's left and how should we sequence it?" | technical-deep-dive |
| "Prep a slide for leadership review" | product-brief |
| "This epic is huge, break it down for me" | initiative-breakdown |

## Typical Workflow

```
Planning:
1. "break down initiative #1234"         → initiative-breakdown maps all the work
2. "do a tech deep-dive on the auth system" → technical-deep-dive analyzes architecture

Communication:
3. "write a product brief for Project X"  → product-brief for stakeholders
4. "summarize this epic for leadership"   → initiative-breakdown (narrative mode)
```

## Output

These skills produce markdown documents. They don't write to a specific location by default; they'll ask where you want the output or present it inline. Common destinations:

- Vault project files (`Projects/{name}.md`)
- Report directory (`reports/`)
- Inline in conversation (for quick answers)

## Assumptions

- Initiatives are tracked as GitHub issues with sub-issues
- Related PRs are linked via GitHub's issue references or PR descriptions
- Code is accessible via the filesystem or `gh` CLI
- Output should be human-readable markdown, not raw data
- These skills are read-only; they analyze and report but don't modify code or issues
