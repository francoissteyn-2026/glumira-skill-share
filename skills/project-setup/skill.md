---
name: project-setup
description: >
  Use when starting a new project from scratch or scoping an existing one for
  Claude Code. Sets up governance, memory, state tracking, hooks, and canonical
  folder structure in the correct order. Do this ONCE at project start — it takes
  15 minutes and prevents every context-loss and drift problem that follows.
  Trigger phrases: "new project", "set up project", "start a project",
  "scope this project", "project setup", "initialise project".
---

# Project Setup — Start and Scope with Claude Code

Do this once at the start of every project. 15 minutes now saves 15 sessions of
re-explaining context later. Every step below is load-bearing.

---

## Phase 0 — Scope the project before touching files (10 minutes)

Before writing a single file, answer these questions in the session. Claude will help
you sharpen them if you're vague:

```
1. What is this project? (One sentence. Product, not vision.)
2. What is the canonical root path on disk?
3. What is the primary stack? (language, framework, deploy target)
4. What are the 3 things Claude must NEVER do in this project?
5. What are the locked artifacts? (files/features that cannot be changed without explicit scoping)
6. Who is the primary audience? (affects voice, content rules, UX decisions)
7. What is the payment/billing rail? (prevents wrong integrations)
8. What does "done" look like for v1? (ship criteria — prevents scope creep)
```

Write these answers down. They become your CLAUDE.md.

**Sparring check before proceeding:** Run `/sparring-partner` on your scope. Ask:
> "What assumption in this scope is most likely to be wrong in 3 months?"

If the answer surprises you, revisit before building.

---

## Phase 1 — Create canonical folder structure

```bash
# Project root — adapt to your stack
mkdir -p .claude/hooks
mkdir -p .claude/commands
mkdir -p .claude/skills
mkdir -p .claude/content-os      # only if content-heavy project
mkdir -p docs/06_Operations/Memory
mkdir -p docs/06_Operations/Sessions
mkdir -p docs/00_MASTER           # locked specs and standards
mkdir -p docs/03_Platform         # technical architecture
mkdir -p docs/04_Brand            # design tokens, voice, identity
mkdir -p docs/05_Business         # strategy, pricing, decisions
```

Folder numbering is deliberate — it controls sort order in file explorers and Obsidian.
Use the same numbers across projects so memory files transfer.

---

## Phase 2 — Create governance files (in this order)

### 2.1 — `CLAUDE.md` (project root)

```markdown
# [Project Name] — Claude Governance
# Read at every session start. All rules are binding.
# Last updated: YYYY-MM-DD

## Project overview
[One paragraph. What this is, who it's for, what problem it solves.]

## Canonical root
`[absolute path]` — ONLY writable path. Never save to Desktop, Downloads, or cloud sync.

## Primary stack
- Framework: [e.g., React 19 + Vite 6]
- Language: [e.g., TypeScript 5.6]
- Deploy: [e.g., Vercel]
- Database: [e.g., Supabase]
- Payment: [e.g., Stripe / XE / none]

## Non-negotiables (LOCKED)
1. [Hard rule 1 — what Claude must never do]
2. [Hard rule 2]
3. [Hard rule 3]
[Keep to 10 or fewer. More than 10 = not non-negotiable, just preferences.]

## Locked artifacts
- [File or feature] — LOCKED. Do not modify without explicit scoping.
[List everything that is "golden standard" — cannot be touched in any redesign pass.]

## Boot protocol (run at every /start)
1. Read this file
2. Read docs/06_Operations/Memory/MEMORY.md + 3 most recent topic files
3. Read docs/06_Operations/STATE.md
4. Read docs/06_Operations/KNOWN_ISSUES.md
5. Confirm understanding, then proceed with task

## Closing protocol (LOCKED)
Every completed task ends with exactly:
1. Continue this workflow
2. Summarize / Save
3. Discard
No variations.

## Session discipline
- Fresh session every 15–20 messages (compact cadence)
- Edit prompts on mistakes — never send follow-up corrections
- One task per session. One commit per session. Stop.

## What we build
[Optional: link to roadmap, v1 scope doc, or next-actions file]
```

### 2.2 — `TRUTH.md` (project root, optional but recommended)

```markdown
# [Project Name] — Single Source of Truth

## Boot order
1. Read this file
2. Read CLAUDE.md
3. Read docs/06_Operations/Memory/MEMORY.md

## Canonical paths
| File | Path |
|------|------|
| Project root | [absolute path] |
| Memory index | [path]/docs/06_Operations/Memory/MEMORY.md |
| Sessions | [path]/docs/06_Operations/Sessions/ |
| State | [path]/docs/06_Operations/STATE.md |
| Known issues | [path]/docs/06_Operations/KNOWN_ISSUES.md |

## Single truths (non-negotiable facts)
| Fact | Value |
|------|-------|
| Deploy platform | [Vercel / Netlify / Railway / etc.] |
| Domain | [your-domain.com] |
| Payment rail | [Stripe / XE / none] |
| Auth provider | [Supabase / Clerk / Auth0 / none] |
| Currency | [USD / GBP / ZAR — pick ONE] |

## Superseded decisions
[Dated entries when a major decision reverses a prior one]
```

### 2.3 — `docs/06_Operations/Memory/MEMORY.md`

```markdown
# Memory Index
# 200-line hard cap. Prune before adding when at limit.

## SESSION START — LOAD THESE FIRST
- **TRUTH.md:** [path] — read this FIRST, then CLAUDE.md
- **Canonical root:** [absolute path]
- After loading, state: "Framework loaded."

## Project Decisions
[Entries added as the project evolves — format: [[YYYY-MM-DD_slug]] — description]

## Feedback & Preferences
[Locked rules, corrections, non-negotiables discovered in sessions]

## Build State
[What's shipped, what's in progress, current sprint]
```

### 2.4 — `docs/06_Operations/STATE.md`

```markdown
# [Project] — Live State
# Updated at session start and at major task transitions.

## Last updated
[YYYY-MM-DD] — project initialized

## Active task
None — project just initialized

## Last completed
- Project governance files created

## Blocked / waiting on founder
- None

## Next concrete step
Choose the first feature to build and run /project-setup Phase 3.

## Session breadcrumb
Check docs/06_Operations/Sessions/ for most recent dated file.
```

### 2.5 — `docs/06_Operations/KNOWN_ISSUES.md`

```markdown
# [Project] — Known Issues
# Rolling list. Add at session end. Mark RESOLVED with date when fixed.

---

## OPEN
[None at project start — add as discovered]

---

## RESOLVED

| ID | Description | Date resolved | Commit/PR |
|----|-------------|---------------|-----------|
```

---

## Phase 3 — Set up Claude Code config

### 3.1 — `.claude/settings.json`

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [{
          "type": "command",
          "command": "[shell] [path-to-hooks]/user-prompt-submit.[sh|ps1]"
        }]
      }
    ],
    "Stop": [
      {
        "hooks": [{
          "type": "command",
          "command": "[shell] [path-to-hooks]/stop-check.[sh|ps1]"
        }]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Write|Edit|NotebookEdit",
        "hooks": [{
          "type": "command",
          "command": "[shell] [path-to-hooks]/pre-tool-use.[sh|ps1]"
        }]
      }
    ]
  }
}
```

See the `state-tracker` skill for hook templates.

### 3.2 — `.claude/commands/start.md`

Create a `/start` boot command. Minimum content:
```markdown
# /start — [Project] Boot

1. Read CLAUDE.md
2. Read MEMORY.md + 3 most recent topic files
3. Read STATE.md + KNOWN_ISSUES.md
4. Confirm framework loaded
5. Proceed with task

Boot report:
1. memory   ✓ / ✗ (BLOCKING)
2. state    ✓ / ~
3. issues   ✓ / ~
```

### 3.3 — `.claude/commands/coins.md`

Create a `/coins` session-end command. See the `closing-protocol` skill for the full template.

---

## Phase 4 — Git setup

```bash
git init
git add CLAUDE.md TRUTH.md docs/06_Operations/
git commit -m "chore: project governance + memory scaffold"
```

Add to `.gitignore`:
```
node_modules/
dist/
build/
.env
.env.local
*.secret
```

---

## Phase 5 — First session discipline

After setup, start the first working session with `/start`. Verify:

- [ ] Boot report shows all steps ✓ or ~ (no ✗ on memory)
- [ ] STATE.md shows correct active task
- [ ] KNOWN_ISSUES.md is empty (no inherited debt)
- [ ] git status is clean before first task

**One task per session from here.** Not two. Not "while I'm in there". One task, one commit, one closer.

---

## Phase 6 — Design tokens (if UI project)

Before writing any component, define:

```json
// design-tokens.json
{
  "color": {
    "primary": "#[hex]",
    "secondary": "#[hex]",
    "accent": "#[hex]",
    "neutral": { "50": "#[hex]", "100": "#[hex]", "900": "#[hex]" },
    "semantic": { "success": "#[hex]", "warning": "#[hex]", "danger": "#[hex]" }
  },
  "spacing": { "base": "8px", "scale": [4, 8, 12, 16, 24, 32, 48, 64] },
  "radius": { "sm": "4px", "md": "8px", "lg": "12px", "full": "9999px" },
  "typography": {
    "display": { "size": "56px", "weight": 700, "lineHeight": 1.1 },
    "h1": { "size": "32px", "weight": 600, "lineHeight": 1.2 },
    "body": { "size": "16px", "weight": 400, "lineHeight": 1.6 }
  }
}
```

Give this file to Claude in every design session. It cannot invent wrong values if the values are always in context.

---

## What this prevents

| Problem | How this setup prevents it |
|---------|---------------------------|
| "I forgot what we decided" | MEMORY.md + dated topic files |
| Claude writing to wrong folder | PreToolUse hook + canonical root in CLAUDE.md |
| Session ends without saving | Stop hook + closing protocol |
| Wrong colour in 47th component | Design tokens file always in context |
| Claude violating locked rules | Non-negotiables in CLAUDE.md at every session |
| Lost context after compaction | STATE.md always current |
| Known bugs re-discovered | KNOWN_ISSUES.md seeded at discovery |
| Scope creep | Non-goals defined at Phase 0, locked in CLAUDE.md |
