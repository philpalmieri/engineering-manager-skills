---
name: interview-analysis
description: |-
  Transform raw behavioral interview notes into a structured summary with rubric
  assessments, recommendation, and self-coaching. Takes raw notes and a scoring
  template, produces a polished write-up preserving the original notes below.
  Use after conducting a behavioral interview.
---

# Interview Analysis

Transform raw interview notes into a structured, paste-ready summary. Scores each question against the rubric, synthesizes a recommendation, and generates interviewer self-coaching for improvement.

## When to use

- "Clean up interview notes", "analyze interview for [candidate]", "interview analysis"
- "Score this interview", "write up my interview notes"
- After conducting a behavioral or values-based interview

## Inputs

The user must provide:

1. **Raw interview notes** — the interviewer's real-time notes. Can be:
   - Pasted directly into the conversation
   - A file path to the notes document
   - Already in a candidate file that needs cleanup

2. **Interview template/rubric** — the question bank and scoring criteria. Can be:
   - A file path to the template
   - Pasted directly
   - Referenced by name if the user has a known templates directory

3. **Target level** (optional) — helps calibrate rubric expectations. Infer from template name if not stated.

4. **Interviewer name** (optional) — for attribution in the summary header. If not provided, omit or ask.

If any required input is missing, ask the user to provide it before proceeding.

## Output Structure

Produce a **clean summary** followed by the **preserved raw notes**:

```markdown
# Behavioral Interview Summary: [Candidate Name] ([Level] Candidate)

**Candidate:** [Name] **Target Level:** [Level] **Interview Type:** Behavioral **Interviewer:** [Name]

**Recommendation: [Hire / No Hire / Lean Hire / Lean No Hire]**

**Overall Impression:** [2-4 sentence synthesis of the interviewer's assessment. Direct and honest.]

**Key Concerns:** (if applicable, bullet list of specific patterns observed)
**Key Strengths:** (if applicable, bullet list of standout positives)

---

## Question N: [Topic Name]

**Q:** _[Full question text from the template, italicized]_

**Values/Competencies:** [Dimensions from the template for this question]

**Summary:** [Narrative paragraph synthesizing the raw notes into coherent prose. Include what was asked, what the candidate said, how follow-ups went, and how the candidate responded to probing. Tell the story of the conversation, not just summarize the answer.]

**Lesson Learned:** [Include ONLY when the question is about failure, feedback, or growth. Extract what the candidate claims to have learned. Omit for other question types.]

**Org Fit Signal:** [Optional. Include when the answer reveals signal about what kind of role, team, or environment the candidate would thrive in. Omit if no clear signal.]

**Rubric Assessment:**

- [✅/⚠️/❌] [Rubric criteria from template]; [brief evidence or gap noted]
- [✅/⚠️/❌] [Next rubric criteria]; [brief evidence or gap noted]
...

---

[Repeat for each question asked]

---

## Candidate's Questions

- [List questions the candidate asked]
- [If the questions reveal signal, add a brief note]

---

## Interviewer Notes

[Synthesis of behavioral observations, communication style, red flags, and patterns. Written in first person from the interviewer's perspective. Professional but direct.]

---

## Interviewer Self-Coaching

For each question asked, note what follow-ups could have been asked to get stronger signal.

### Question N: [Topic]
- **Missing signal on [area].** A good follow-up would have been: _"[suggested question]"_ This would test [what it would reveal].

### General Notes
- [Overall interviewing observations: pacing, depth vs. breadth, time management]

---
---

# Raw Interview Notes

[Original raw notes preserved exactly as written]
```

## Rubric Assessment Rules

For each question, pull rubric criteria from the template (both **Positives** and **Negatives** columns if available). Assess each positive criterion:

- **✅** Strong positive signal: candidate provided specific, concrete evidence
- **⚠️** Mixed or partial signal: candidate touched on this but lacked specifics or depth
- **❌** Negative signal or absent: candidate did not address this, or actively demonstrated the corresponding negative

Format: `- [emoji] [Criterion from template]; [your specific evidence or observation]`

Use semicolons (not colons or dashes) to separate criterion from evidence. When a candidate's answer matches a negative from the rubric, call that out explicitly.

## Determining Recommendation

Read the raw notes for explicit hire/no-hire signals from the interviewer. If stated, use that. If not explicit, derive from rubric assessments:

- Majority ✅ with no ❌ on critical criteria = **Hire** or **Lean Hire**
- Mixed ✅ and ⚠️ = **Lean Hire** or **Lean No Hire** depending on severity
- Any ❌ on critical criteria (depth of expertise, decision-making, learning from failure) = **Lean No Hire** or **No Hire**
- Multiple ❌ or clear behavioral red flags = **No Hire**

## How to Identify Questions Asked

Match questions in raw notes against the template's question bank by content/topic. Note which template question number each corresponds to. If a question doesn't match the template, include it as supplemental.

## Follow-Up Conversation Flow

The Summary for each question should document the full conversation arc:
- What the candidate said initially
- What follow-up questions were asked
- How the candidate responded to probing (deeper, redirect, repeat, struggle?)
- Whether hints or reframes were needed
- Editorial observations from raw notes woven in naturally

## Handling the Raw Notes

- Preserve raw notes **exactly as written** below the double separator (`---\n---`)
- Do not fix typos or grammar in the raw notes section
- The summary section is the polished version; raw notes are the source of truth

## Rules

- **Self-contained.** Do not reference other candidate files as examples or calibration.
- **Always generate the Self-Coaching section.** For each question, suggest 1-2 follow-ups that could have yielded stronger signal. Reference specific rubric criteria that were hard to assess. Be constructive and specific, not generic.
- **Preserve existing content.** If writing to a file, raw notes stay intact below the separator.
- **Be honest in assessments.** Don't grade on a curve. If signal is weak, say so.
- **Output flexibility.** If the user doesn't specify a file path, present the analysis in the conversation. If they specify a path, write it there.
