# Context Check & Rebuild Skill

**Purpose:** Validate session context awareness and catch environment drift before it causes problems

**When to Use:**
- Behavior seems off (not using known skills, wrong paths, forgetting conventions)
- After long breaks (1+ week between sessions)
- Proactively before complex multi-step work
- When making basic mistakes about known facts

**What It Checks:**
- File system structure (Obsidian vault, team data, skills directory)
- Skills availability and symlink health
- Data structures (team roster, monthly activity data)
- Custom instructions loaded correctly
- Recent session history

**Output:**
- Health check report with ✅/⚠️/❌ status per area
- Verification tests (asks known questions to validate awareness)
- Specific fixes for any issues found
- Recommendations for next steps

**The Problem This Solves:**

Context can be **loaded but not actively used**. The agent might have:
- Custom instructions in the prompt ✅
- Skills symlinked ✅
- Data files present ✅

But still:
- Claim things don't exist without checking ❌
- Make assumptions instead of verifying ❌
- Rush to "fix" things that aren't broken ❌

This skill forces explicit validation: "Do I actually know what I think I know?"

**Usage:**
- `run context check` — full health report
- `verify environment` — same thing
- `rebuild context` — check + fix issues

Use proactively when starting big tasks or reactively when behavior drifts.
