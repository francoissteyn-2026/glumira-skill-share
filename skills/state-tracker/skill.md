---
name: state-tracker
description: >
  Use when starting a new project or when a project is losing context across sessions.
  Implements two files borrowed from adversarial audit of AI Agent Memory Scaffold
  (Nina Dnj, 2026-05-14): STATE.md (live task state) and KNOWN_ISSUES.md (rolling
  open issues). Adapted for hook-enforced systems -- not passive markdown convention.
  Trigger phrases: "add state tracking", "set up state", "I'm losing context between
  sessions", "add known issues tracking", "borrow the scaffold pattern".
---

# State Tracker — STATE.md + KNOWN_ISSUES.md

Two files. Borrowed from adversarial audit of Nina Dnj's AI Agent Memory Scaffold.
Adapted for enforcement-based systems (hooks, harness). Not copy-pasted.

## Why these two, not all seven

| Scaffold file | Why skipped |
|---|---|
| AGENTS.md | Covered by CLAUDE.md or equivalent project governance |
| PROJECT.md | Covered by MEMORY.md or project README |
| DECISIONS.md | Covered by dated memory topic files |
| HANDOFF.md | Covered by session summaries in Sessions/ |
| WORKLOG.md | Covered by git log + session ledger |

STATE.md and KNOWN_ISSUES.md are the only two with no equivalent in a typical harness setup.

---

## File 1 — STATE.md

**Purpose:** Single file that always reflects where the project is right now. The "pick up here" signal for the next session or agent. Written at session end, read at session start.

**Location:** `<project-root>/docs/06_Operations/STATE.md` (or `<root>/STATE.md` for smaller projects)

**Template:**

```markdown
# [Project] — Live State
# Updated at every session start and at major task transitions.

## Last updated
YYYY-MM-DD — <one-line description of last session>

## Active task
<What is currently being worked on, or "None — session ended cleanly">

## Last completed
- <item 1>
- <item 2>

## Blocked / waiting on [owner]
- <blocker 1> — <who needs to act>
- <blocker 2>

## Next concrete step
<Single sentence. The first thing the next session should do.>

## Session breadcrumb
If this session ended mid-task, check Sessions/ for the most recent dated file.
```

**When to update:**
- At session start: read it, confirm or correct the state
- At major inflection points during a session (direction change, blocker found, feature shipped)
- At session end (before /coins or equivalent): overwrite with final state

---

## File 2 — KNOWN_ISSUES.md

**Purpose:** Rolling list of open problems. Issues discovered in one session that aren't fixed yet. Prevents known problems from getting buried in session history and re-discovered three sessions later.

**Location:** `<project-root>/docs/06_Operations/KNOWN_ISSUES.md` (or `<root>/KNOWN_ISSUES.md`)

**Template:**

```markdown
# [Project] — Known Issues
# Rolling list. Add at session end. Mark RESOLVED with date when fixed.
# Owned issues only -- not a wishlist, not a roadmap.

---

## OPEN

### [CATEGORY-NNN] Short title
Description of the issue. What breaks, what the impact is.
**Impact:** <user-visible / CI / security / launch blocker>
**Fix:** <one-line description of what the fix requires>

---

## RESOLVED

| ID | Description | Date resolved | Commit/PR |
|----|-------------|---------------|-----------|
| [CAT-001] | Description | YYYY-MM-DD | abc1234 |
```

**ID conventions:**
- `STRUCT-` — file/folder structure problems
- `BUILD-` — build, test, or CI failures
- `CONTENT-` — copy, clinical, or content errors
- `INFRA-` — deployment, DNS, credentials
- `SEC-` — security issues
- `UX-` — user-facing bugs

**When to update:**
- Add new issues: whenever a problem is identified during a session
- Mark resolved: immediately when the fix is committed — not "later"
- Review at session start: skim for anything that affects the current task

---

## Wiring into hooks (Claude Code)

### SessionStart hook — auto-read both files

Add to your `session-start.ps1`:

```powershell
$context += "Second action before any task: Read docs/06_Operations/STATE.md "
$context += "and docs/06_Operations/KNOWN_ISSUES.md. "
$context += "STATE.md shows live task state. KNOWN_ISSUES.md shows open problems. "
$context += "Mark ~ degraded in boot report if either is missing."
```

### /start command — add to step 3 (memory)

Add to the memory step in your boot command:
> Also read `STATE.md` (live task state) and `KNOWN_ISSUES.md` (open issues) — mark `~` if either missing.

### /coins or session-end command — add update steps

```
Step N — STATE.md update (always)
Overwrite STATE.md: set Active task, Last completed, Blocked, Next concrete step.

Step N+1 — KNOWN_ISSUES.md update (if applicable)
New issues found: add under OPEN with ID.
Issues resolved: move to RESOLVED table with date + commit.
```

### Stop hook — surface open blockers (optional)

If your harness has a Stop hook, add a check:
```powershell
$openIssues = Select-String -Path "docs/06_Operations/KNOWN_ISSUES.md" -Pattern "### \[" |
              Where-Object { $_ -notmatch "RESOLVED" }
if ($openIssues.Count -gt 0) {
    Write-Output "  Open issues: $($openIssues.Count) in KNOWN_ISSUES.md"
}
```

---

## Implementation checklist for a new project

```
[ ] Create STATE.md from template at <root>/docs/06_Operations/STATE.md
[ ] Create KNOWN_ISSUES.md from template at <root>/docs/06_Operations/KNOWN_ISSUES.md
[ ] Add STATE.md read to SessionStart hook or boot script
[ ] Add both files to /start or equivalent boot command (step 3 - memory)
[ ] Add update steps to /coins or session-end command
[ ] Populate KNOWN_ISSUES.md with any issues already known
[ ] Set STATE.md to reflect current project state
[ ] Commit both files
```

---

## What this does NOT replace

- Git history — still the ground truth for what changed
- Session summaries — still the narrative record of what happened
- MEMORY.md / CLAUDE.md / project governance — still the rules
- Enforcement hooks — still the only reliable way to ensure compliance

STATE.md and KNOWN_ISSUES.md are read-layer improvements, not enforcement.
They make context available. Hooks make rules stick.
