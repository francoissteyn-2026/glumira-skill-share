---
name: superpowers
description: >
  Use when the user says "superpowers", "@superpowers", "/superpowers",
  "spec this", "plan this", "go deep", "research X thoroughly", "5-agent",
  "parallel execution", or when a task needs PhD-grade depth across multiple
  specialist domains simultaneously. Spawns 5 parallel specialists in one message.
  Do NOT use for single-file edits or simple tasks — overhead is not worth it.
---

# Superpowers — 5-Agent Parallel Execution

Compress timeline. Never compress quality. Spawn 5 specialists simultaneously,
each with isolated context, each owning a distinct workstream.

**Core principle:** Fresh subagent per domain + independent context + synthesis pass = quality that one Claude cannot match alone.

---

## When to use

```
Task crosses ≥3 specialist domains?  →  Use superpowers
Need PhD-level depth in <30 min?     →  Use superpowers
Single-file edit or clear task?       →  Use standard execution (GSD)
Need multi-session persistence?       →  Use Ruflo instead
```

---

## The 5-agent pattern

Spawn all 5 in one message (parallel fan-out). Never sequential.

```
Agent(name: "architect",    prompt: <architect brief>, run_in_background: true)
Agent(name: "implementer",  prompt: <implementer brief>, run_in_background: true)
Agent(name: "reviewer",     prompt: <reviewer brief>, run_in_background: true)
Agent(name: "ux-auditor",   prompt: <ux brief>, run_in_background: true)
Agent(name: "domain-expert",prompt: <domain brief>, run_in_background: true)
```

Wait for all 5 to return. Synthesize. Output.

---

## Default role set (adapt to task)

| Slot | Default role | Owns |
|------|-------------|------|
| 1 | **Architect** | System design, data flow, module boundaries, trade-offs |
| 2 | **Implementer** | Writes the actual code — spec-compliant, tested, committed |
| 3 | **Reviewer** | Checks output against project constraints (CLAUDE.md, golden standards, locked rules) |
| 4 | **UX / Product** | Mobile-first layout, interaction design, accessibility, conversion |
| 5 | **Domain Expert** | The specialist the task actually needs — security, clinical, performance, legal, etc. |

Adapt to the task. A data pipeline task might swap UX for a data-engineer. A legal document might swap implementer for a technical-writer. The pattern is the constant; the roles flex.

---

## Spawn prompt template

Every agent brief must include:

```
ROLE: <slug>
TASK: <specific deliverable — not "help with X", but "produce Y as a file at Z">
CONSTRAINTS: Read <path/to/CLAUDE.md or equivalent> before starting.
  Key constraints verbatim:
  - <constraint 1>
  - <constraint 2>
NON-GOALS: <what is explicitly out of scope — prevents scope creep>
OUTPUT: Deliver via the return value of this agent call. Do not ask questions — make a decision and note it.
QUALITY BAR: Production-ready or do not output. This is not a draft.
```

**Never share context between agents.** Each gets exactly what it needs — no more. Context pollution is the #1 failure mode.

---

## Activation pattern

1. **Decompose** — break the task into 5 parallel workstreams. Each workstream must be independently executable (no dependencies between them at execution time).
2. **Brief** — write the spawn prompt for each. Verbatim constraints, specific deliverable, explicit non-goals.
3. **Spawn** — all 5 in one message. `run_in_background: true` on every call.
4. **Collect** — wait for all returns. Read each agent's output.
5. **Synthesize** — you (the orchestrator) produce the final unified output. The synthesis is the value-add over just running one Claude.
6. **Commit** — one commit, one summary, one closer.

---

## Two-stage review (subagent-driven variant)

When executing an implementation plan with superpowers, add a review stage after each task:

1. **Implementer** executes the task
2. **Spec reviewer** confirms output matches the spec (not just "looks right")
3. **Quality reviewer** checks code quality, style, test coverage

Only mark a task complete after both reviews pass. This is slower per task but eliminates the rework cycle that costs 3× more tokens downstream.

---

## What superpowers is NOT

- Not a replacement for a clear plan (write the plan first, then spawn)
- Not for simple tasks (overhead is ~15–20k tokens per session)
- Not for tasks with hard sequential dependencies (use standard execution)
- Not for multi-session persistence (use Ruflo for that)

---

## Superpowers + Design Council

For design decisions specifically, superpowers can feed Design Council:

1. **Superpowers** runs the research phase — 5 agents explore the solution space in parallel
2. **Design Council** debates the top options — 9 specialists cross-examine the superpowers findings
3. **GSD execution** ships the winner

This three-layer stack is reserved for decisions with significant irreversible consequences.
