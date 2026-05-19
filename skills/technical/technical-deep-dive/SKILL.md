---
name: technical-deep-dive
description: |-
  Generate a detailed technical analysis of a project from actual code, schema, issues,
  and architecture. Covers how things work today, what's done, what's remaining, risk
  assessment, and sprint-level sequencing. Use for engineering planning, architecture
  reviews, or onboarding engineers to a complex initiative.
license: MIT
---

# Technical Deep Dive

Generate a thorough technical analysis of a project or initiative by exploring the actual codebase, schema, issues, and engineering commentary. This is the engineering complement to the Product Brief: it's for people who need to understand how things work, what's built, and what's left.

## When to use

- "Do a deep dive on [project]", "technical analysis of [initiative]", "what's the engineering state of [project]?"
- When planning sprint work and you need a realistic picture of remaining effort
- When onboarding an engineer to a complex existing project
- When preparing for an architecture review or technical decision

## Prerequisites

- GitHub issues/epics tracking the project
- GitHub CLI (`gh`) authenticated with org access
- Access to relevant source repos for code exploration
- Vault project files, ADRs, and prior breakdowns (if they exist)

## Process

### Step 1: Gather context from GitHub

```bash
# Main tracking issue
gh issue view {number} --repo {org}/{repo} --json title,body,comments,labels,assignees

# Sub-issues
gh search issues --repo {org}/{repo} --json number,title,state,labels,assignees,body "label:{project-label}"

# Recent and historical PRs
gh search prs --owner={org} --json number,title,repository,state,mergedAt,author "{project keyword}"

# Discussions or ADRs (if in a discussions-enabled repo)
gh api "repos/{org}/{repo}/discussions" --jq '.[].title' 2>/dev/null
```

Read issue bodies and comments. Engineering decisions, context, and "why we did it this way" often live in comments, not the issue description.

### Step 2: Explore the codebase

Launch parallel explore agents to investigate different aspects simultaneously. For a typical project, this might mean:

1. **Schema and data model**: Find the database tables/models, understand relationships, identify scale constraints
2. **Code paths and architecture**: Trace the main flows, identify integration points, map feature flags
3. **Test coverage**: What's tested? What's not? Are there integration tests or just unit tests?
4. **Cross-service boundaries**: If the project spans multiple services, map the interaction points

Give each agent specific questions to answer, not just "look at the code." For example:
- "Find the schema for X table. What are the indexes? What's the primary key strategy?"
- "Trace the flow from [entry point] to [storage]. What jobs are involved? What can fail?"
- "List all feature flags related to [project]. Which are enabled in production?"

### Step 3: Build the analysis

Structure the deep dive:

#### Overview
2-3 paragraph summary for someone who wants the executive version. What is this, where does it stand, what are the key risks?

#### How It Works Today (The Actual Code)

**The domain/package structure:**
Where does this code live? Key files and their purposes. Use a table:

| File/Path | Purpose |
|-----------|---------|
| `path/to/model.rb` | ActiveRecord model, PK strategy, key methods |
| `path/to/job.rb` | Background job, what triggers it, what it does |

**The schema:**
Actual table definitions. Column names, types, indexes. Not what the docs say, what the migration says.

**The flow:**
ASCII diagram of the main path(s). Show where data enters, what processes it, where it's stored.

```
Input → Service A → Job Queue → Service B → Storage
                         │
                    Error Path → Dead Letter → Alert
```

**Feature flags:**
Which flags control this feature? What's their current state?

#### What's Done (Scorecard)

Create a concrete scorecard of all sub-tasks or migration items:

| Item | Status | PR/Issue | Notes |
|------|--------|----------|-------|
| Basic case | ✅ Done | [#123](url) | Shipped 2026-03 |
| Edge case 1 | 🔄 In progress | [#456](url) | PR open, needs review |
| Edge case 2 | ⬜ Not started | [#789](url) | Blocked on decision X |

Count: X done, Y in progress, Z remaining.

#### Key Technical Decisions

For each open or recently-made decision:
- What's the question?
- What are the options? (with tradeoffs)
- What's the current leaning and why?
- What would change the answer?

#### Workstream Breakdown

Group remaining work into parallel workstreams:

**Workstream 1: [Name]**
- What it covers
- Dependencies
- Estimated complexity (small/medium/large)
- Who's working on it or who should

**Workstream 2: [Name]**
- Same structure

#### Sprint Sequencing

If the user needs a delivery plan:

| Sprint | Workstream | Deliverables | Dependencies |
|--------|-----------|-------------|-------------|
| Current | WS1 | Items A, B | None |
| Next | WS1, WS2 | Items C, D | A must ship first |

#### Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| [Specific risk] | Med | High | [What we'd do] |

Be honest about unknowns. "We don't know X yet" is a valid risk.

### Step 4: Save and present

Save to the vault's Projects directory (or Breakdowns subdirectory).

Summarize for the user:
- Top 3 findings (surprises, risks, or opportunities)
- What you couldn't determine from code/issues alone (needs human context)
- Suggested next actions

## Exploration strategy

The quality of a deep dive depends on how well you explore the code. Guidelines:

- **Parallelize aggressively.** Launch 3-5 explore agents with distinct, focused questions. Don't have one agent try to understand everything.
- **Follow the data.** Start from the schema/model layer and trace upward. Understanding the data model usually reveals the architecture.
- **Read the tests.** Tests often document edge cases and expected behaviors that aren't written anywhere else.
- **Check git blame for recent changes.** If a file was recently modified, it's likely relevant to current work.
- **Read issue comments, not just descriptions.** The real decisions and context live in the discussion.

## Rules

- **Code over docs.** If the docs say one thing and the code says another, the code is right. Note the discrepancy.
- **Be precise about what you verified vs. inferred.** "The schema has 3 columns" (verified) vs. "This probably handles X" (inferred).
- **Don't pad.** If you couldn't find information about something, say so. Don't fill gaps with speculation.
- **Include actual file paths and code references.** Someone should be able to open the files you reference.
- **Credit the engineers.** Use @mentions for who built what, based on PR authors and issue assignees.
- **Date the analysis.** Code changes. Note when the exploration was done so the reader knows the freshness.
