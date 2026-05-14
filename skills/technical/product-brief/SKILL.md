---
name: product-brief
description: |-
  Generate a product-oriented summary of a project or initiative from GitHub data.
  Covers the problem, solution, complexity, user value, what's shipped, what's in progress,
  and what needs product attention. Use when ramping up a PM, presenting to leadership,
  or creating a shareable project overview.
---

# Product Brief

Generate a product-oriented summary of a project or initiative. Designed to give someone with zero context a clear understanding of what the project does, why it's hard, what's shipped, and what still needs work. Written for a product or leadership audience, not a purely engineering one.

## When to use

- "Write a product brief for [project]", "summarize [project] for Rohan", "create a project overview"
- When onboarding a PM or stakeholder to an existing initiative
- When you need a shareable doc that explains a project beyond the engineering details

## Prerequisites

- A GitHub issue, epic, or initiative tracking the project
- GitHub CLI (`gh`) authenticated with org access
- Vault project files (if any exist) for additional context

## Process

### Step 1: Gather raw material

**From GitHub:**
```bash
# The main tracking issue/epic
gh issue view {number} --repo {org}/{repo} --json title,body,comments,labels,assignees,createdAt

# Sub-issues and linked work
gh search issues --repo {org}/{repo} --json number,title,state,labels,assignees "label:{project-label}"

# Recent PRs related to the project
gh search prs --owner={org} --json number,title,repository,state,mergedAt "{project keyword}"
```

**From the vault:**
- Check `Projects/` for existing project files, breakdowns, or ADRs
- Check recent dailies and People files for context, decisions, and discussion notes

**From code (if deeper understanding needed):**
- Launch explore agents to read relevant code, schema, and architecture
- Focus on: how does it actually work today? What's the real complexity?

### Step 2: Organize into sections

Build the brief in this order:

#### The Problem
What's broken, missing, or inadequate today? Explain it so someone outside the team understands why this matters. Be concrete: "here's what happens today" → "here's why that's bad."

Don't lead with the solution. Start with the pain.

#### What [Project] Does
The solution, explained clearly. What changes? How does it work at a high level? Avoid implementation details here; save those for the technical deep dive.

#### Why It's Complicated
The honest version. What makes this harder than it sounds? Multiple code paths? Scale constraints? Cross-team dependencies? Design decisions that aren't settled? This section builds credibility and sets expectations.

Include sub-sections for distinct sources of complexity if needed.

#### What's Already Shipped
Concrete deliverables that are done. For each:
- What it is (one line)
- Why it mattered
- Current status (GA, staff-only, behind flag, etc.)

#### What's In Progress Now
Active work. For each workstream:
- What's being built and by whom
- Current state (PR links, issue links)
- Key decisions still open

#### Why This Matters to Users
Translate engineering work into user value. Group by audience:
- **Persona 1**: what they get, why they care
- **Persona 2**: different angle
- Keep it grounded in real use cases, not abstract "better security"

#### The Product Story
What's the narrative for announcing this? Not marketing copy, but the kernel of the story. What would a blog post headline say? What's the "before/after" that resonates?

#### What Needs Product Work
Explicitly call out where product decisions are needed:
- Naming and positioning
- Rollout strategy
- Public vs. internal scope
- UX decisions
- Pricing or packaging implications

### Step 3: Add references

End with a structured references section:

```markdown
## References

### Tracking
- [Initiative](url) — top-level epic
- [Sub-epic 1](url) — description

### Key Milestones
**Done:**
- [Issue/PR](url) — what it delivered

**Outstanding:**
- [Issue](url) — what's left, current status

### Design & Documentation
- [ADR](url) — topic (last updated: YYYY-MM-DD)
- [Discussion](url) — topic
```

Flag stale docs (>6 months since last update) so the reader knows what might be outdated.

### Step 4: Save and present

Save to the vault's Projects directory (or a Breakdowns subdirectory if the team uses one).

Present a summary to the user highlighting:
- Key narrative choices you made
- Anything you're uncertain about (conflicting info, stale sources)
- Gaps where you couldn't find data

## Writing guidelines

- **Assume the reader has no context.** Don't reference internal shorthand without explaining it.
- **Be honest about complexity.** Don't minimize hard problems. The brief builds trust by showing you understand what's actually hard.
- **Separate "what's done" from "what's claimed."** If an issue says "shipped" but you can't find the PR, note that.
- **Use plain language.** This isn't an RFC. Write it like you're explaining to a smart person over coffee.
- **Include real links.** Every claim should have a traceable reference (issue, PR, discussion, code path).

## Rules

- **No speculation presented as fact.** If you're inferring something, say so.
- **Credit people.** Use @mentions for who built what.
- **Don't editorialize the product story.** Present the narrative options; let the PM choose.
- **Flag conflicts.** If the issue description says one thing and the code says another, call it out.
- **This is a living doc.** Note the generation date and suggest when it should be refreshed.
