---
name: closing-protocol
description: >
  Enforces the three-option closing protocol at the end of every completed task.
  LOCKED — no variations. Trigger: automatically at task completion.
  Never skip. Never add a fourth option. Never substitute prose.
  Trigger phrases: "closing protocol", "task complete", "done", "finished",
  or any workstream that produces a deliverable.
---

# Closing Protocol — Three-Option Task Closer

Every completed task ends with exactly this. No variations. No extra prose after it.

```
1. Continue this workflow
2. Summarize / Save
3. Discard
```

---

## What each option means

| Option | Action |
|--------|--------|
| **1. Continue this workflow** | Keep executing in this session — next step, next task, next iteration |
| **2. Summarize / Save** | Write session summary, update STATE.md, commit, compact |
| **3. Discard** | Abandon this workstream without saving — reverse any changes if applicable |

---

## Rules (non-negotiable)

1. **Show it ONCE** when the entire workstream is complete — not after every sub-task
2. **Multi-part task:** A + B + C → show the closer after C, not after A or B
3. **Blocked:** If blocked on the human (credentials, decision, external action needed), flag the block, THEN show the closer
4. **Never skip it.** A completed task without the closer forces the human to ask for it — that's a wasted message
5. **Never substitute** with "let me know what you'd like to do" or any prose equivalent
6. **Never add a fourth option** unless explicitly instructed for a specific task

---

## What "Summarize / Save" triggers

When the user picks option 2, run this sequence:

### Step 1 — Estimate tokens
```
Messages × ~500
Tool calls × ~1,000
File reads × (lines × 5)
Subagents spawned × ~10,000
session_coins = total_tokens ÷ 1,000
```

### Step 2 — Write session summary
`docs/06_Operations/Sessions/YYYY-MM-DD_HHMM_session.md`:
```markdown
# Session Summary — YYYY-MM-DD HH:MM

**Coins this session:** N
**Cumulative:** total

## Accomplished
- [item]

## Files changed
- [file] — [change]

## Outstanding (carry forward)
🔴 [blocked/urgent]
🟡 [in progress]
🔵 [next up]

## Next concrete step
[Single sentence]
```

### Step 3 — Update STATE.md
```markdown
## Last updated
YYYY-MM-DD — [one-line description]

## Active task
None — session ended cleanly  (or: [task name] — blocked on [owner])

## Last completed
- [item]

## Blocked / waiting on [owner]
- [blocker] — [who acts]

## Next concrete step
[Single sentence]
```

### Step 4 — Commit
```bash
git add docs/06_Operations/Sessions/ docs/06_Operations/STATE.md
git commit -m "docs: session summary YYYY-MM-DD"
```

---

## Installation

### Global (all projects, all sessions)

Add to `~/.claude/CLAUDE.md`:
```markdown
## Closing protocol — MANDATORY

Every completed task ends with exactly this:
1. Continue this workflow
2. Summarize / Save
3. Discard

Show once when the workstream is complete. Never skip. Never vary.
```

### Per-project

Add the same block to your project's `CLAUDE.md`.

### Skill hook

Add to your `/start` command:
```
Closing protocol: LOCKED. Every completed task ends with:
1. Continue this workflow / 2. Summarize / Save / 3. Discard
No variations. No extra prose. Show once at workstream end.
```

---

## Why this exists

Without a structured closer, every task ends with one of:
- "Let me know if you'd like me to continue" — forces a reply
- No closer at all — the human doesn't know if the task is done
- An open-ended offer — invites scope creep

The three-option closer makes the session state explicit. Option 1 keeps momentum.
Option 2 forces the save discipline. Option 3 gives the human an explicit exit.
Three options cover 100% of states without requiring prose.
