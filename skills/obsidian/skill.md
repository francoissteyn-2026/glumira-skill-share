---
name: obsidian
description: >
  Use when working with Obsidian vaults — creating wikilinks, navigating memory,
  writing session summaries, maintaining the MEMORY.md index, or setting up the
  canonical vault structure for a new project.
  Trigger phrases: "link vault", "obsidian", "wikilink", "update memory",
  "canonical structure", "set up obsidian", "new project structure".
---

# Obsidian × Claude Code Integration

Obsidian as the project's permanent memory layer. Claude Code reads and writes to the vault.
No plugins required for basic operation — just file conventions.

---

## Vault configuration (recommended)

The cleanest pattern: **the project repo IS the Obsidian vault.**

```
your-project/          ← git root AND vault root
├── .obsidian/         ← Obsidian config (gitignored or committed)
├── CLAUDE.md          ← Project governance (auto-loaded by Claude Code)
├── TRUTH.md           ← Single source of truth pointer (optional)
├── docs/
│   └── 06_Operations/
│       ├── Memory/
│       │   ├── MEMORY.md          ← Index (200-line max)
│       │   └── YYYY-MM-DD_*.md    ← Topic files
│       ├── Sessions/
│       │   └── YYYY-MM-DD_*.md    ← Session summaries
│       ├── STATE.md               ← Live task state
│       └── KNOWN_ISSUES.md        ← Rolling open issues
└── src/
```

This means every code file is a vault node. Decision files link to code files.
Code changes surface in the Obsidian graph automatically.

---

## Wikilink rules (LOCKED)

```markdown
✅  [[2026-05-14_slug]]           ← date + slug, no extension
✅  [[MEMORY]]                    ← exact filename, no extension
❌  [[path/to/file.md]]           ← path syntax doesn't render as wikilink
❌  file:///C:/project/file.md    ← never clickable in Claude Code
```

For links shared in Claude Code chat (clickable):
```markdown
[label](docs/06_Operations/Memory/2026-05-14_slug.md)  ← relative path only
```

Absolute paths do NOT click-open in the Claude Code VS Code extension.

---

## Memory file structure

Every decision, feedback, or key finding gets a dated topic file:

```markdown
---
name: Short title — one line
description: What was decided and why — one or two sentences
type: project | feedback | session
---

[Body — 50–200 words. Detail only. Index entry goes in MEMORY.md.]
```

After creating: add one line to `MEMORY.md` in the correct section:
```markdown
- [[YYYY-MM-DD_slug]] — Short description (~150 chars max)
```

**MEMORY.md has a 200-line hard cap.** Prune before adding when at limit.

---

## MEMORY.md sections

| Section | What belongs |
|---------|-------------|
| **SESSION START — LOAD THESE FIRST** | Critical files to read at boot |
| **Project Decisions** | Architecture, feature decisions, platform choices |
| **Feedback & Preferences** | Corrections, style preferences, non-negotiables |
| **Build State** | What's complete, what's pending |

---

## Canonical vault structure for a new project

Run this block when starting a new project with Obsidian integration:

```bash
# Create the full canonical structure
mkdir -p docs/06_Operations/Memory
mkdir -p docs/06_Operations/Sessions

# Root governance files
touch CLAUDE.md
touch TRUTH.md

# Operations files
touch docs/06_Operations/Memory/MEMORY.md
touch docs/06_Operations/STATE.md
touch docs/06_Operations/KNOWN_ISSUES.md
```

Then populate from the templates in the `state-tracker` skill (STATE.md and KNOWN_ISSUES.md).

---

## Canonical placeholder files for new builds

Copy and fill these at project start:

### `CLAUDE.md` starter
```markdown
# [Project Name] — Claude Governance
# Read at every session start. All rules are binding.

## Project overview
[What this project is. One paragraph.]

## Canonical root
`[absolute path]` — the only writable path.

## Non-negotiables
1. [Locked rule 1]
2. [Locked rule 2]
...

## Boot protocol
1. Read this file
2. Read docs/06_Operations/Memory/MEMORY.md + 3 most recent topic files
3. Read docs/06_Operations/STATE.md
4. Read docs/06_Operations/KNOWN_ISSUES.md
5. Proceed with task

## Session close protocol
Every completed task ends with:
1. Continue this workflow
2. Summarize / Save
3. Discard
```

### `TRUTH.md` starter
```markdown
# [Project Name] — Source of Truth

## Boot order
1. Read this file
2. Read CLAUDE.md
3. Read docs/06_Operations/Memory/MEMORY.md

## Canonical paths
- Project root: [absolute path]
- Memory index: [path]/docs/06_Operations/Memory/MEMORY.md
- Sessions: [path]/docs/06_Operations/Sessions/
- State: [path]/docs/06_Operations/STATE.md

## Single truths
- Deploy target: [platform]
- Primary stack: [stack]
- Payment rail: [provider]
- [Other critical single truths]
```

### `MEMORY.md` starter
```markdown
# Memory Index

## SESSION START — LOAD THESE FIRST
- **TRUTH.md:** [path] — read this FIRST
- **Canonical root:** [absolute path]
- After loading, state: "Framework loaded."

## Project Decisions
[First entries go here as the project evolves]

## Feedback & Preferences
[Locked rules and corrections go here]

## Build State
[What's shipped, what's in progress]
```

---

## Session protocol

**Session start:**
1. Read `MEMORY.md`
2. Read the 3 most-recently-modified topic files in Memory/
3. Read `STATE.md`
4. Read `KNOWN_ISSUES.md`

**Session end (with /coins or equivalent):**
1. Write `Sessions/YYYY-MM-DD_HHMM_session.md`
2. If new decisions: write topic file + update `MEMORY.md`
3. Overwrite `STATE.md` with current state
4. Update `KNOWN_ISSUES.md` if new issues or resolutions
5. Commit all four categories in one commit

---

## Useful Obsidian plugins for this pattern

| Plugin | Why |
|--------|-----|
| `obsidian-git` | Auto-commit vault changes; keeps git history and vault in sync |
| `dataview` | Query your memory files as a database (`TABLE date, description FROM "docs/06_Operations/Memory"`) |
| `calendar` | Navigate session summaries by date |
| `local-rest-api` | Lets Claude Code write to vault via API (advanced) |
