---
name: interview-prep
description: |-
  Generate a structured behavioral interview script from a candidate's resume and
  an interview rubric/template. Selects questions targeting the candidate's specific
  experience, builds resume-specific follow-ups, and explains question selection rationale.
  Use when preparing for a behavioral or values-based interview.
license: MIT
---

# Interview Prep

Generate a tailored behavioral interview script by analyzing a candidate's resume against an interview template/rubric. Produces question selection, resume-specific follow-ups, and strategic rationale.

## When to use

- "Prepare interview", "prep interview for [candidate]", "interview prep"
- "Build an interview script for this resume"
- Before any structured behavioral interview

## Inputs

The user must provide (via paste, file path, or URL):

1. **Resume** — the candidate's resume or profile. Can be:
   - Pasted directly into the conversation
   - A file path to a document (PDF, markdown, text)
   - A LinkedIn URL or other profile source

2. **Interview template/rubric** — the question bank and scoring criteria. Can be:
   - A file path to a template document
   - Pasted directly
   - Referenced by name if the user has a known templates directory

3. **Target level** (optional) — the role level being interviewed for. If not provided, infer from the template name or ask.

4. **Interview duration** (optional) — defaults to 30 minutes if not specified.

If any required input is missing, ask the user to provide it before proceeding.

## Process

### Step 1: Analyze the Resume

Identify:
- **Career arc:** companies, roles, tenure, progression
- **Core technical domains:** what systems, technologies, and problem spaces they've worked in
- **Level-appropriate signals:** architecture ownership, cross-team leadership, mentoring, org-wide impact (calibrate expectations to the target level)
- **Specific claims worth probing:** metrics, scope claims, buzzword-heavy areas, gaps
- **Red flags or areas to pressure-test:** all-wins resume, vague ownership language ("contributed to," "participated in"), dramatic metrics without context

### Step 2: Select Questions (3-4 from the template)

Choose questions that:
- **Target the candidate's specific experience.** Match questions to resume hooks (if they claim cross-team alignment wins, pick the collaboration question; if they list a major production system, pick the technical depth question).
- **Cover diverse values/competencies.** Aim for breadth across the rubric's evaluation dimensions.
- **Include at least one character question.** Always include one of: Biggest Failure, Tough Feedback, Biggest Conflict, or similar self-awareness questions. These test honesty and are harder to fake.
- **Probe the resume's weakest spots.** If the resume is all-wins, pick a failure question. If it's heavy on individual contribution, pick a collaboration question.

For a 30-minute interview:
- 5 min intros
- 3-4 questions at 6-8 min each (with follow-ups)
- 5 min candidate questions

Depth over breadth. 3-4 questions is the target.

### Step 3: Build Resume-Specific Follow-Ups

For each selected question, write 2-3 follow-up probes that are **specific to this candidate's resume**. These are not generic follow-ups from the template; they reference actual claims, projects, or metrics from the resume.

Good follow-ups:
- Reference a specific resume claim and ask the candidate to go deeper
- Test whether a metric is real ("4% improvement in accuracy; how did you measure that?")
- Probe ownership language ("you say you 'led teams of 2-5'; what did leading look like day to day?")
- Ask for the failure/challenge side of a success story

Bad follow-ups:
- Generic versions of the template follow-ups
- Yes/no questions
- Questions that let the candidate stay at the surface

### Step 4: Write the "Why These Questions" Section

For each question, write a brief justification explaining:
- What resume hook triggered this question
- What signal you're looking for
- What a strong vs. weak answer would reveal

Also include a competency/value coverage summary showing which evaluation dimensions each question maps to.

## Output Structure

```markdown
# Interview Plan: [Duration] Minutes

Template: [Template Name]

> **Candidate Name** / Title, ~N yrs experience
> Company history (most recent first, with tenure)
> Strengths: [3-5 key themes from resume]

---
**Intros:**
-

## Suggested Flow

### N. ⏱ N min: Q[template#]: [Topic Name]
> Hook: [1-2 sentences explaining why this question targets this candidate's resume. Reference specific projects, claims, or metrics.]

**[Full question text from the template, bolded]**

Follow-ups to probe:
- [Resume-specific follow-up 1]
- [Resume-specific follow-up 2]
- [Resume-specific follow-up 3, if needed]

[Repeat for each question]

---

## Candidate Questions
-

---

## Why These Questions

**Q[N] ([Topic]):** [2-3 sentences explaining the strategic reason for this question given this candidate's profile.]

[Repeat for each question]

**Value/Competency Coverage:**
- [Dimension]: [Which questions cover it]
```

## Handling Edge Cases

- **If a candidate file already has content:** Preserve existing content and prepend or merge the script. Do not overwrite notes.
- **If the resume is thin:** Note this in the candidate brief and lean toward questions that give the candidate room to demonstrate depth rather than questions that require breadth.
- **If the candidate's experience doesn't map well to any template question:** Note the gap and suggest a custom question that fits their background, clearly marking it as supplemental.
- **If the template is missing:** Ask which template or rubric to use before proceeding.
- **If no output path is specified:** Present the script in the conversation. If the user specifies a file path, write it there.

## Rules

- **Self-contained.** Do not reference other candidate files as examples.
- **Resume-specific, not generic.** Every follow-up should reference something from THIS candidate's background.
- **Depth over breadth.** 3-4 well-chosen questions beat 6 surface-level ones.
- **Character matters.** Always include at least one self-awareness/failure question.
- **Preserve existing content.** If writing to a file that already has notes, merge rather than overwrite.
