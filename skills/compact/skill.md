---
name: compact
description: >
  Engages session-efficiency discipline: fresh session every 15-20 messages,
  edit-not-followup on mistakes, always end with a next-session summary.
  Trigger phrases: "compact", "session efficiency", "fresh session",
  "save context", "15-20 cadence", "context discipline".
  Also auto-activates at boot to maintain discipline throughout the session.
---

# Compact — Session Efficiency Discipline

Long Claude Code sessions are expensive. A fresh session can reduce context cost by up to 98%
vs. a very long thread. This skill enforces the discipline that prevents token waste.

---

## The cadence rule

**Start a fresh session every 15–20 messages.**

Not a suggestion — a discipline. At message 15, or when a major task completes,
surface the natural breakpoint.

Why 15–20 and not more:
- Context cost grows with conversation length
- Stale context from earlier in the session can contaminate later decisions
- Fresh sessions get better answers on fresh topics
- Your working memory (the session) should match your task scope

---

## What compact discipline looks like

### At boot
When this skill activates, set an internal message counter. Track it.

### At message ~15 or major task completion
Surface this:
```
── Context checkpoint ──────────────────────────────
Session: ~15 messages. Good point to start fresh.
Current task status: [complete / in progress / blocked]
Next session starts with: [one concrete sentence]
────────────────────────────────────────────────────
```

Only surface this if a natural stopping point exists. Do not interrupt mid-task.

### On mistakes — edit, don't follow up

When Claude makes an error:
- **Do NOT** send a follow-up message correcting it
- **Do** edit the previous prompt directly and resubmit
- This keeps the context clean and prevents the error from becoming part of the permanent context

### Before ending a session

Always write a closing summary before `/compact` or starting a new chat:

```markdown
## Next session starts with
[Single sentence — the first thing the next session should do]

## Outstanding
- [Item 1]
- [Item 2]

## Files changed this session
- [file] — [what changed]
```

---

## When to use /compact mid-session

Use `/compact` proactively if:
- Context window indicator is in the warning zone
- The session has produced heavy file reads (design docs, long code files)
- A major task just completed and the next task is unrelated
- You're about to start a complex multi-file operation

Do not wait until the context is full — compact at 70–80% to keep response quality high.

---

## Heavy-output sessions

When a session produces heavy outputs (PDF generation, long documents, test suites):
- Consider a dedicated tab for that generation task
- Keep your main session clean for coordination
- The generation tab can be closed when done

---

## The token cost reality

| Session length | Approximate context cost |
|---------------|--------------------------|
| 10 messages | Baseline |
| 20 messages | ~2× |
| 40 messages | ~4-5× |
| 80 messages | ~10-15× |
| 160+ messages | ~20-40× (severe degradation) |

Long sessions don't just cost more — they produce worse output. Stale context from 60 messages
ago contaminates every response. Fresh sessions are faster AND cheaper AND more accurate.

---

## Wiring into boot

Add to your `/start` command or `SessionStart` hook:
```
Compact discipline: ON. Fresh session every 15–20 messages.
At major task completion, surface the context checkpoint.
Edit prompts on mistakes — never follow up.
Always end with "Next session starts with:" summary.
```
