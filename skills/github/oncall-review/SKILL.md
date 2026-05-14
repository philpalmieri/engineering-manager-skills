---
name: oncall-review
description: |-
  Score the quality of an on-call or first-responder shift based on issue completeness,
  handoff quality, and engagement signals. Use when reviewing a completed on-call shift
  or generating team metrics.
---

# On-Call Quality Review

Evaluate the quality of a completed on-call (or first-responder) shift by scoring against a rubric. Produces a star rating and qualitative assessment.

## When to use

- "How was this week's on-call?", "rate the FR shift", "on-call quality check"
- As part of periodic team health reviews

## Prerequisites

- A GitHub issue tracking the on-call shift (typically auto-created with a checklist template)
- Access to the issue via `gh` CLI
- A scoring rubric stored in the vault (location from setup config)

## Process

### Step 1: Fetch the shift issue

```bash
gh issue view {number} --repo {org}/{team-repo} --json title,body,comments,labels,assignees,createdAt,closedAt
```

Extract:
- **Assignee**: who was on-call
- **Checklist items**: count total, count checked
- **Comments**: count total, assess quality (see below)
- **Handoff**: look for a handoff comment or section

### Step 2: Score against rubric

Apply the scoring rubric. A reasonable default rubric:

| Criterion | Points | How to assess |
|-----------|:------:|---------------|
| Checklist completion ≥ 20% | +1 | Count checked / total checkboxes |
| Checklist completion ≥ 50% | +1 | Same |
| Comments ≥ 3 | +1 | Count non-bot comments |
| Comments ≥ 6 | +1 | Same |
| Handoff completed | +1 | Look for a handoff comment, section, or linked handoff issue |

**Maximum score: ⭐⭐⭐⭐⭐**

Teams should customize this rubric to match their expectations. Store it in the vault (e.g., `Areas/Team Metrics/On-Call Quality Rubric.md`).

### Step 3: Qualitative assessment

Beyond the raw score, assess:

- **Investigation quality**: Did comments show actual debugging (Splunk queries, Sentry links, SLO analysis)? Or just "acknowledged, auto-resolved"?
- **Checklist engagement**: Were unchecked items skipped because they didn't apply, or because they were missed?
- **Handoff quality**: Does the handoff give the next person enough context to start without spelunking? Or is it "nothing happened this week"?
- **Incident response**: If incidents fired, were they triaged promptly? Is there follow-up documented?

### Step 4: Output

Write a summary (to the vault or inline, depending on context):

```markdown
### On-Call Review: @{assignee} ({date_range})

**Score: ⭐⭐⭐** (3/5)

| Criterion | Result |
|-----------|--------|
| Checkboxes ≥ 20% | ✅ (47%) |
| Checkboxes ≥ 50% | ❌ (47%) |
| Comments ≥ 3 | ✅ (3) |
| Comments ≥ 6 | ❌ (3) |
| Handoff | ⚠️ Partial |

**Notes:**
- Investigation quality was above average (detailed SLO breakdown with monitoring links)
- Checkbox completion hurt by [reason if identifiable]
- Handoff was [complete/partial/missing]: [specifics]
```

If maintaining a tracking doc, append this to the historical record.

## Rubric customization

The default rubric above is a starting point. Teams should adjust based on:

- **Shift type**: On-call vs. first-responder may have different expectations
- **Team size**: Smaller teams may have fewer incidents and lower comment baselines
- **Maturity**: New teams should start with lower thresholds and raise them over time

Store the active rubric in the vault so the team can see and iterate on it.

## Rules

- **Score the shift, not the person.** The review assesses the quality of the work product, not the individual. Frame feedback constructively.
- **Account for quiet weeks.** A low score during a genuinely quiet week (no incidents, no alerts) should be noted as context, not penalized.
- **Flag data issues.** If the issue body appears truncated (common with long checklists), note which criteria couldn't be assessed.
- **Be honest.** A 2-star shift is a 2-star shift. Don't inflate scores to be nice, but do explain what would have made it better.
