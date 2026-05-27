---
name: context-check
description: |-
  Validate and rebuild session context awareness. Checks that key paths, skills,
  data structures, and configurations are accessible and properly indexed. Use
  when behavior seems off, after long breaks, or proactively before complex tasks
  to ensure full environment awareness.
license: MIT
---

# Context Check & Rebuild

Validate that the agent has proper awareness of your environment, skills, data structures, and workflows. This is a bootstrap/reset skill: it reads your actual configuration files and proves the agent has loaded them correctly.

## When to Trigger

- "check context", "rebuild context", "verify environment"
- "something seems off", "you're not using my skills", "did you forget X"
- Proactively before quarterly reviews, big analysis runs, or multi-day projects
- After long breaks (1+ week between sessions)
- When the agent makes basic mistakes about known paths, skills, or conventions

## What This Skill Checks

### 1. File System Structure
- Obsidian vault path and key directories (Dailies, Projects, People, etc.)
- Team data directory and month structure
- Skills directory and symlinks
- Session state folder

### 2. Skills Availability
- Count of linked skills in `~/.copilot/skills/`
- Verify key skills exist (team-data-sync, bus-factor, review-network, etc.)
- Check for broken symlinks

### 3. Data Structures
- Team roster (`data/team.json`)
- Available data months in `data/YYYY-MM/`
- Sample PR/review files to validate structure
- Recent activity data exists

### 4. Custom Instructions & Preferences
- Vault path from custom instructions
- Writing style preferences
- Known workflows (prep, snippet, 1:1s)

### 5. Recent Session History
- Checkpoint availability
- Recent work topics
- Files modified in last session

## Process

### Step 1: Run Health Checks

Verify each area by actually reading files and directories (not from memory):

```markdown
## Environment Health Check

### 📁 File System
- {✅/❌} Obsidian vault: {path from custom instructions} (accessible?)
- {✅/❌} Team data: {data directory} (found?)
- {✅/⚠️} Skills directory: {N} skills, {N} broken symlinks
- {✅/❌} Session state: current session active

### 🛠️ Skills Availability
- {✅/❌} Core skills loaded: {N} total
- {✅/❌} Metrics skills: {list discovered metrics skills}
- {✅/❌} GitHub skills: {list discovered github skills}
- {✅/❌} Orchestration: {list discovered orchestration skills}
- {⚠️} Broken: {list any broken symlinks}

### 📊 Data Availability
- {✅/❌} Team roster: {N} members loaded
- {✅/❌} Data months: {earliest} through {latest} ({N} months)
- {✅/❌} Latest data: {month} (all members present?)
- {✅/❌} Sample PR data: validated structure

### 👥 Team Roster (Active Context)
{Read team.json and output the full roster table here — see Step 3}

### 📝 Custom Instructions
- {✅/❌} Obsidian vault path: {configured path}
- {✅/❌} Writing style: {summary of preferences}
- {✅/❌} Workflows: {list known workflows}
- {✅/❌} Fiscal year calendar: {if configured}

### 🕐 Session Context
- Session ID: {id}
- Checkpoints: {N}
- Recent work: {brief summary}
```

### Step 2: Identify Issues

For each ⚠️ or ❌, provide:
- What's wrong
- Impact (what might break)
- How to fix

### Step 3: Load and Validate Team Roster

**Critical:** Actually READ `team.json` and load team member details into active context. Do not rely on memory or cached information.

For each member, extract and confirm:
- login (GitHub username)
- name (full name)
- pronouns
- vault_name (Obsidian People file reference, if configured)
- level, joined date
- relationship (direct, skip-level, peer, etc.)

Output the full roster table so it's in the immediate conversation context:

```markdown
### Team Roster (Loaded from team.json)

| Login | Name | Pronouns | Vault | Level | Joined |
|-------|------|----------|-------|-------|--------|
| {login} | {name} | {pronouns} | {vault_name} | {level} | {joined} |
| ... | ... | ... | ... | ... | ... |
```

**Also load collaborators** (manager, peers, skip-levels) if present in the roster with their pronouns and relationships.

### Step 4: Validate Other Key Files

Read sample files from critical areas to confirm access and structure:
- One daily note from Obsidian (most recent)
- One PR data file from latest month
- One skill SKILL.md
- Confirm structures match expectations

### Step 5: Rebuild Indexes (if needed)

If skills or data seem stale:
- Re-scan skills directory
- Re-read team roster
- Confirm data month availability
- Rebuild internal index

### Step 6: Verification Test

Use the data loaded in Step 3 to build a self-test. Pick facts FROM the actual team.json and environment to verify the agent can answer correctly:

```markdown
### Verification (Generated from team.json)

| Question | Expected (from file) | Actual (from context) | Status |
|----------|---------------------|----------------------|--------|
| Obsidian vault path? | {from custom instructions} | {what agent believes} | {✅/❌} |
| How many team members? | {count from team.json} | {what agent believes} | {✅/❌} |
| {member_1}'s pronouns? | {from team.json} | {what agent believes} | {✅/❌} |
| {member_2}'s login? | {from team.json} | {what agent believes} | {✅/❌} |
| What metrics skills exist? | {from skills dir scan} | {what agent believes} | {✅/❌} |
| Latest data month? | {from data dir scan} | {what agent believes} | {✅/❌} |
```

**Important:** Don't just check "does team.json exist?" — actually load it and prove you can answer specific questions about team members using the real data.

## Output Format

```markdown
# Context Check Report

**Status:** {🟢 Healthy | 🟡 Minor Issues | 🔴 Major Problems}  
**Run at:** {timestamp}

---

## Health Check Results

{detailed checks from Step 1}

---

## 🔴 Critical Issues (if any)

### Issue: {description}
- **Impact:** {what breaks}
- **Fix:** {specific action}

---

## 🟡 Warnings (if any)

### Warning: {description}
- **Impact:** {potential issues}
- **Fix:** {specific action}

---

## Verification Tests

{dynamically generated from actual data — see Step 6}

---

## Recommendations

{If 🟢 Healthy:}
- Context is solid. Ready for complex tasks.

{If 🟡 Minor Issues:}
- [Specific fixes for warnings]
- Re-run context-check after fixing

{If 🔴 Major Problems:}
- [Critical fixes needed]
- Consider fresh session after repairs
```

## Integration with Better Verification Discipline

**Before making claims, check facts:**
- Don't say "skill X doesn't exist" → run `ls ~/.copilot/skills/` first
- Don't say "path Y is wrong" → run `view` on it first
- Don't say "format Z is incorrect" → read comparison files first

**When asked to verify/validate:**
1. Read the actual files FIRST
2. Compare to what you think you know
3. Report discrepancies honestly
4. Then answer the question

**When making changes:**
1. Check if the thing you're "fixing" actually needs fixing
2. Read the current state before editing
3. Compare to peer files for patterns
4. Validate assumptions before acting

## Proactive Usage

**When to run context-check:**
- **First thing** in a new session after 1+ week break
- **Before** quarterly team health reports
- **After** major skill updates or data structure changes
- **When** the user says "something seems off"
- **If** you notice yourself making basic mistakes

**When to auto-load team.json (without full context-check):**

Even if you don't run the full context-check, **always load team.json** before:
- Writing about team members (snippets, 1:1 notes, reviews)
- Analyzing team data (bus factor, velocity, review network)
- Referencing anyone by name
- Creating any communication involving the team

**Why:** Having team.json in your prompt ≠ actively using it. You need to READ it and put the details in immediate context before writing about people. This prevents:
- Wrong pronouns
- Mixing up logins and display names
- Forgetting who reports to whom
- Using incorrect vault references

**Pattern to follow:**
```
User: "prep my snippet"
You: [Read team.json first] → [Load member details] → [Then draft snippet with correct pronouns/names]

Not:
You: [Draft snippet from memory] → [Get details wrong] → [User corrects you]
```

## Self-Check Triggers

If you (the agent) notice yourself:
- Claiming a path doesn't exist without checking
- Saying "I don't have X" without verifying
- Making assumptions about structure without reading files
- Contradicting recent known facts

→ Stop and suggest: "Let me run a context check to make sure I have accurate environment awareness."

## Key Takeaway

**Context exists in the prompt, but awareness requires active verification.**

Having instructions loaded ≠ using them correctly. This skill forces explicit validation of:
- What you claim to know
- What's actually there
- Whether your internal model matches reality

Run it when behavior drifts from expectations.
