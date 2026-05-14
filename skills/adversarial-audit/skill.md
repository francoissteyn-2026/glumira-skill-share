---
name: adversarial-audit
description: >
  Use when evaluating any external system, skill, framework, stack, scaffold, or tool
  before adopting it. Attacks from multiple angles — gaps, redundancies, contradictions,
  security, performance, fit — to produce a per-component verdict before anything gets
  installed or borrowed.
  Trigger phrases: "adversarial audit", "audit this", "evaluate this before I adopt it",
  "stress-test this system", "audit my stack", "should I use this", "what's wrong with this".
  Also use on your OWN stack periodically — the audit finds drift before users do.
---

# Adversarial Audit

Evaluate any system, skill, framework, or tool by attacking it before adopting it.
Find the gaps, redundancies, and failure modes. Produce a per-component verdict.

**The rule:** benefit of the doubt is a liability. Every component earns its place or gets cut.

---

## When to invoke

| Trigger | Example |
|---------|---------|
| Evaluating an external framework | Nina Dnj's AI Agent Memory Scaffold, a prompt library, a boilerplate |
| Adding a new tool to your stack | New MCP server, Claude skill, npm package |
| Auditing your OWN system | Quarterly stack review, before a major refactor |
| Someone posts "here's how I do X" | Evaluate before copying |
| Pre-launch hardening | Stress-test the system before it's live |
| Post-incident review | What failed and why |

---

## The five attack angles

Run all five on every component being audited. No skipping.

### 1. Gap attack — What is this solving that you don't already solve?

```
Q: What problem does this component address?
Q: Is that problem already addressed in the existing stack?
Q: If so — how? Is the existing solution worse?
Q: If the problem isn't addressed — how serious is the gap?
   - P0: Causes active failures (data loss, security breach, context loss)
   - P1: Causes waste or rework (repeated mistakes, re-discovered bugs)
   - P2: Nice to have (convenience, speed)
   - P3: No measurable impact
```

### 2. Redundancy attack — What does this duplicate?

```
Q: Does this component overlap with anything already in the stack?
Q: What is the exact overlap — identical, partial, or adjacent?
Q: If there is overlap — which version is better? By what criterion?
Q: Does adding this create two sources of truth for the same thing?
   Two sources of truth = guaranteed drift = guaranteed contradiction.
```

### 3. Contradiction attack — What does this conflict with?

```
Q: Does any rule in this component contradict a rule in the existing system?
Q: Does this component assume a workflow that the existing system doesn't support?
Q: If adopted wholesale — what would you have to REMOVE from the existing stack?
Q: Is the removal cost worth the adoption benefit?
```

### 4. Enforcement attack — What happens when this is ignored?

```
Q: Is this component enforced mechanically (hooks, CI, linting) or by convention?
Q: If by convention only — how many sessions until it drifts?
Q: What is the failure mode when it IS ignored?
Q: Does the adopter have the infrastructure to enforce it?
   Convention without enforcement = expensive documentation, not a system.
```

### 5. Fit attack — Is this solving YOUR problem or someone else's?

```
Q: What context was this built for? (team size, stack, domain, scale)
Q: What context are YOU in?
Q: What assumptions does this component make that don't hold in your context?
Q: What would you need to change to make it fit — and does the changed version
   still solve the original problem?
```

---

## Verdict format

After running all five attacks on each component, assign a verdict:

```
ADOPT   — Take it wholesale. Minimal modification needed. Genuine gap filled.
BORROW  — Take the concept, not the implementation. Adapt to your context.
SKIP    — Already covered, redundant, or wrong fit. Not worth the overhead.
WATCH   — Interesting but not ready. Revisit when [specific condition].
FIX     — Problem is real but this solution is wrong. Build your own instead.
```

---

## Full audit output format

```markdown
# Adversarial Audit — [Subject]
Date: YYYY-MM-DD
Auditor: [Claude / human / council]
Subject: [URL, repo, description]

## Context loaded
[What was read before auditing — the subject + your existing stack governance]

## Component verdicts

### [Component name]
**Purpose:** [What it claims to do]
**Gap attack:** [Gap found / Already covered by X]
**Redundancy attack:** [Overlaps with Y / No overlap]
**Contradiction attack:** [Conflicts with Z rule / No conflict]
**Enforcement attack:** [Convention only / Mechanically enforced via X]
**Fit attack:** [Built for team of 20 / Fits solo SaaS]
**Verdict:** ADOPT / BORROW / SKIP / WATCH / FIX
**Rationale:** [One sentence. Why this verdict and not the next-best alternative.]
**If BORROW — what specifically to take:** [The concept, pattern, or template — not the file]
**If FIX — what to build instead:** [One-line description]

[Repeat for each component]

## Summary

| Component | Verdict | Action |
|-----------|---------|--------|
| [name] | ADOPT | Install as-is |
| [name] | BORROW | Adapt pattern into [existing file] |
| [name] | SKIP | Already covered by [X] |

**Net result:** [N] adopted, [N] borrowed, [N] skipped
**Estimated implementation time:** [X hours]
**Risk if adopted as-is:** [Low / Medium / High — one sentence reason]
```

---

## Auditing your OWN stack (periodic review)

Run this quarterly or after any major session that produces new rules:

### Self-audit questions

```
1. Are there rules in CLAUDE.md that Claude consistently ignores?
   → If yes: the rule is not enforced. Add a hook or remove the rule.

2. Are there files in the memory system that haven't been read in 30 days?
   → If yes: prune or archive them. Stale memory = noise.

3. Are there skills installed that haven't been invoked in 10 sessions?
   → If yes: remove them. Unused skills add context overhead.

4. Are there two files that say the same thing in slightly different ways?
   → If yes: merge or delete one. Two sources of truth = drift.

5. Is there a known issue in KNOWN_ISSUES.md that has been open for 30+ days?
   → If yes: either fix it this session or explicitly defer with a date.

6. Are there hooks that aren't firing correctly?
   → Verify by checking exit codes. Silent hook failures = no enforcement.

7. Has any "locked" artifact been modified without explicit scoping?
   → Check git log for the locked files. Unauthorised changes = governance failure.
```

### Self-audit output

```markdown
# Stack Self-Audit — YYYY-MM-DD

## Ignored rules (no mechanical enforcement)
- [Rule] in [file] — [how many times violated in git log] — Action: [add hook / remove rule]

## Stale memory files (not read in 30 days)
- [file] — [last referenced] — Action: [prune / archive]

## Unused skills (not invoked in 10 sessions)
- [skill] — Action: [remove]

## Duplicate sources of truth
- [file A] and [file B] say the same thing — Action: [merge into X]

## Long-open known issues (30+ days)
- [ISSUE-ID] — open since [date] — Action: [fix this session / defer to YYYY-MM-DD]

## Enforcement failures
- [hook] not firing — [error] — Action: [fix]

## Locked artifact violations
- [none / commit hash + file + what changed]
```

---

## Real example: Nina Dnj AI Agent Memory Scaffold audit (2026-05-14)

**Subject:** 7-file memory scaffold — AGENTS.md, PROJECT.md, DECISIONS.md, HANDOFF.md, WORKLOG.md, STATE.md, KNOWN_ISSUES.md

| Component | Verdict | Rationale |
|-----------|---------|-----------|
| AGENTS.md | SKIP | Covered by CLAUDE.md (single source of truth for agent rules) |
| PROJECT.md | SKIP | Covered by MEMORY.md + project README |
| DECISIONS.md | SKIP | Covered by dated memory topic files in Memory/ |
| HANDOFF.md | SKIP | Covered by session summaries in Sessions/ |
| WORKLOG.md | SKIP | Covered by git log + session ledger (coins.md) |
| STATE.md | BORROW | No enforced equivalent existed. Live task state was only in session context — lost on compact. |
| KNOWN_ISSUES.md | BORROW | Known bugs were buried in session history and re-discovered 3 sessions later. |

**Net result:** 0 adopted, 2 borrowed, 5 skipped.
**What was borrowed:** The STATE.md and KNOWN_ISSUES.md templates — adapted to GluMira's folder structure, wired into SessionStart hook and /start boot command, added to /coins session-end sequence.
**What was NOT borrowed:** The other 5 files — because equivalent coverage already existed. Adding them would create duplicate sources of truth.

---

## Why "adversarial" and not just "review"

A review looks for what's good. An adversarial audit assumes everything is wrong until proven otherwise.

The difference in practice:
- **Review:** "This looks useful for managing decisions."
- **Adversarial audit:** "DECISIONS.md is redundant with my dated topic files. Adding it creates two places decisions are stored. Two places = drift within 3 sessions. SKIP."

Adversarial audits find the redundancy. Reviews miss it because they're looking for value, not problems.
