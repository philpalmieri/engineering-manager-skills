---
name: initiative-breakdown
description: |-
    Deep-dive breakdown of a GitHub initiative or epic issue. Fetches the issue,
    all sub-issues, related PRs, comments, milestones, and linked work to produce
    a comprehensive project summary with narrative depth. Output includes: project
    history and origin story, what's been done (with context on why it mattered),
    what's outstanding (grouped by delivery/logical chunk with stakes and sequencing),
    architecture concerns, priority recommendations, and key links.

    Use when asked to break down, summarize, or analyze an initiative, epic, or
    large body of tracked work on GitHub.
license: MIT
---

# Initiative Breakdown Skill

## When to Use

Use this skill when the user asks to:
- Break down an initiative or epic issue into a summary
- Understand a project's history, current state, and remaining work
- Get a "what's done / what's left" view of a large body of work
- Analyze delivery breakpoints and architectural concerns for a project

## Inputs

The user will provide:
1. **A GitHub issue URL or number** — the initiative/epic to analyze. If a full URL is given, extract the owner, repo, and issue number. If only a number is given, ask which repo.
2. **Optional: output path** — where to save the breakdown. If not specified, write to the current working directory as `{Issue Title} Breakdown.md`. If the user specifies a path, use that. If the user says "on screen" or "just show me," print to stdout instead of creating a file.

## Process

### Step 1: Fetch the Initiative Issue

```bash
gh issue view {number} --repo {owner}/{repo} --json title,body,labels,assignees,state,comments,createdAt
```

Extract:
- Problem statement and definition of done
- Team (DRI, engineering, product, design)
- Milestones and corresponding work links
- All comments (weekly status updates, discussions)
- Labels (used in Step 2 to find related issues)

### Step 2: Fetch All Sub-Issues and Related Work

From the issue body, comments, and labels, gather related work:

**a) Parse linked issues:** Scan body and comments for issue references (`owner/repo#NNNN`, `#NNNN`, full URLs). Fetch each one.

**b) Search by label:** If the issue has project-specific labels, search the same repo for all issues with that label:
```bash
gh search issues "repo:{owner}/{repo}" --label {label} --limit 100 --json number,title,state,createdAt,closedAt
```

**c) Search by milestone:** If milestones are referenced (especially cross-repo), fetch those too.

**Cap detection:** If any search returns exactly 100 results, split the date range and re-query to avoid API caps.

For each sub-issue found, fetch: title, state, body (first 300-500 chars), created/closed dates, labels.

### Step 3: Fetch Related PRs

- Cross-reference PR numbers mentioned in issue comments and sub-issue bodies
- If the user has local team data (e.g., `data/YYYY-MM/{login}-prs.json`), check there first
- Otherwise use `gh` CLI to look up specific PRs mentioned

### Step 4: Map the Timeline

From comments (especially automated status reports), build a timeline:
- When did the project start?
- Key milestones and target dates (note any shifts)
- Trend changes (on-track → at-risk → blocked)
- Who has been providing updates?

### Step 5: Categorize Work

Group all discovered issues into:
- **✅ Done** — closed issues, merged PRs
- **🔴 In Progress / Critical** — open, actively worked, blocking partners or deadlines
- **🟡 Planned / Next** — open, designed or scoped but not yet started
- **🔵 Future / Nice-to-Have** — open, no urgency signal, stretch goals

Use labels, status fields, and comment recency to determine category.

### Step 6: Identify Delivery Breakpoints

Group related work into logical deliveries:
- Each delivery should be independently shippable
- Note dependencies between deliveries
- Flag blocked items and what's blocking them
- Name each delivery descriptively (not just "Phase 1")

### Step 7: Analyze Architecture & Concerns

Look for signals in issue descriptions, comments, and PR titles:
- Performance bottlenecks or scalability concerns
- Dependencies on other teams, services, or external partners
- Tech debt items (migrations, deprecated code, schema changes)
- Design decisions that are still open or contentious
- Risk patterns (timeline slips, scope creep, partner dependencies, key-person risk)

### Step 8: Write the Breakdown

Use this output structure:

```markdown
## Project Summary

[3-4 paragraphs that tell the STORY, not just the facts:]

[Paragraph 1 — The Problem: What real-world problem does this solve? Frame it from the
user/customer perspective. What was broken, missing, or painful before this project existed?
Use a concrete example if possible ("An org builds a container image, pushes it to three
registries, deploys it across two clusters — and today there's no way to answer...")]

[Paragraph 2 — The Solution: What does this project actually do? Be specific about the
mechanism, not just the category. Name the API shape, the data model, the integration
pattern. Help the reader build a mental model of how it works.]

[Paragraph 3 — The Players: Who uses this and why? Name each partner/consumer with a
one-sentence explanation of their specific use case and what they get out of it. This is
where "who cares about this" becomes concrete.]

[Paragraph 4 (optional) — The Stakes: Why does this matter strategically? Is it a
platform bet, a partner commitment, a competitive response? What happens if it fails
or stalls?]

**Initiative:** [link]
**Original Epic:** [link if separate]
**DRI:** @person
**Label:** `label-name`

---

## How It Started

[Don't just list dates — tell the origin story:]

[Opening paragraph: What triggered this project? Was it a partner request, a strategic
bet, an organic evolution from something else? Name the original vision and how it
differs from where the project is today. If the project pivoted or grew beyond its
original scope, say so.]

**Key partners driving this work:**
1. **[Partner Name]** — [2-3 sentences: what they want, why they care, what they
   contribute. Include tension if relevant — are they pushing timeline? Blocking on
   decisions? Largest consumer?]
2. **[Partner Name]** — [Same depth]

**Timeline:**
- [Date]: [Event — include context, not just "issue opened". Say WHY it opened, what
  triggered it]
- [Date]: [Key decision or pivot point]
- Current: [Where things stand right now, in a full sentence]

---

## Architecture Overview

[Open with a paragraph explaining the high-level architecture BEFORE the diagram.
Help the reader understand what they're about to see. Name the key components and
their roles. Call out the most important dependency or integration point.]

```
[ASCII diagram showing system topology, data flow, or component relationships]
```

**Key dependencies:**
- **[System/Service]:** [What it does for this project, why it matters, what happens
  when it's slow/down. This is where you flag coupling risks.]
- **[System/Service]:** [Same depth]

[If there are important architectural decisions that shaped the current design, mention
them here with brief rationale — "X was chosen over Y because..."]

---

## What Has Already Been Done

**Total: [N] closed issues**

[Group completed work into named categories that tell the story of the project's
evolution. Each category should have a brief narrative intro (1-2 sentences) explaining
what this body of work accomplished and why it mattered, THEN the issue/PR list.]

### ✅ [Category Name]

[1-2 sentence narrative: What problem did this batch solve? What was the state before
and after?]

- [Issue/PR link]: What was done, by whom if notable
- [Issue/PR link]: What was done — call out interesting details, not just titles

[Repeat for each logical category. Good categories tell a story: "Permission System
Overhaul" is better than "Auth stuff". "Partner-Driven Scalability Fixes" is better than
"Performance".]

---

## Major Outstanding Work

**[N] open issues**

### 🔴 Delivery N: [Descriptive Name] ([M] issues)

[Opening paragraph: What does this delivery accomplish? Why is it prioritized at this
level? Who is it blocking and what's the consequence of delay? This paragraph should
make someone who knows nothing about the project understand why this matters.]

| Issue | Title | Status |
|-------|-------|--------|
| [link] | Title | Open/In Progress/Blocked |

**Current state:** [What's actively happening — name the PR, the person, the stage.
"PR #NNNN is open with N comments, currently in review" is better than "in progress".]

**Pipeline:** [If there's a sequence, spell it out: "A must land before B, then C
gates on partner validation, then D is the public ship"]

**Risks/blockers:** [Be specific. "Partner hasn't validated" is better than "partner risk"]

[Repeat for each delivery, using 🔴/🟡/🔵 to signal urgency]

---

## Key Architectural Concerns

[Each concern gets a named sub-heading and a full paragraph explaining the problem,
the evidence, and the impact. Don't just name the concern — explain WHY it's a concern
and what could go wrong if it's not addressed.]

### N. [Concern Name]

[Full paragraph: What's the issue? What evidence from the codebase, issues, or PRs
supports this? What's the blast radius if this bites? Is anyone working on it?
Reference specific issues/PRs that demonstrate the concern.]

[Repeat for each concern]

---

## Delivery Priority (Recommended)

[Ordered list with rationale. Each item should explain WHY it's in this position,
not just what it is:]

1. **[Delivery Name]** — [Why this is first: who it unblocks, what deadline it serves,
   what risk it mitigates]
2. **[Delivery Name]** — [Same depth]

---

## Stats

| Metric | Value |
|--------|-------|
| [Meaningful metrics — total issues, open/closed, PRs by DRI, partner count,
  project duration, team size. Include whatever tells the story of the project's
  scale and health.] |

---

## Key Links

| Resource | Link |
|----------|------|
[Table of important references: epics, ADRs, docs, dashboards, partner comms]
```

## Output

**Default:** Save to `{cwd}/{Issue Title} Breakdown.md` (sanitize the title for filesystem-safe naming).

**If the user specifies a path:** Use that path exactly.

**If the user asks for screen output:** Print the markdown to stdout, do not create a file.

After writing, summarize key findings conversationally: what's the project about, what's the biggest risk, what's the recommended next action.

## Tips for Quality

- **Tell the story, don't just report facts.** Every section should answer "why does this matter?" not just "what happened." A reader should understand the project's trajectory, tensions, and stakes from the breakdown alone.
- **Comments are gold.** Status update comments often contain the best context about decisions, blockers, and timeline shifts. Don't skip them. Quote or paraphrase key insights.
- **Name names and link links.** Say "@user shipped the constraint fix in PR #429277" not "a fix was merged." Specificity builds trust in the analysis.
- **Look for patterns.** Multiple bug fixes in the same area suggest architectural issues worth calling out. Repeated timeline slips in comments tell a story about scope or complexity.
- **Cross-reference team data.** If available (e.g., team.json or local data files), map GitHub logins to names and roles for readability.
- **Follow the links.** Issues often reference related epics, ADRs, or external docs. Chase at least one level deep. The best context is often one link away from the issue body.
- **Don't just list — analyze.** The value is in grouping, prioritizing, and identifying concerns, not just repeating issue titles. A flat list of 40 issues is useless; 8 named deliveries with narrative context is actionable.
- **Be honest about gaps.** If information is missing (empty issue bodies, no status updates, shell issues with no content), call it out explicitly. "This area has 6 open issues but no recent activity or comments" is useful signal.
- **Respect the hierarchy.** Initiatives contain epics, epics contain batches, batches contain tasks. Understand which level you're analyzing and scope accordingly.
- **Frame partner dynamics.** If external partners are driving the work, explain the relationship: who wants what, what they're waiting on, what leverage they have. This context is critical for prioritization.
- **Surface the human story.** Key-person risk, team size vs. scope, burnout signals (one person authoring 200+ PRs on a project), organizational dynamics — these matter as much as the technical details.
