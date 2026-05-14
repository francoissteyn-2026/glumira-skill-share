---
name: ultrareview
description: >
  Multi-agent parallel code review. DIY equivalent of Claude Code's /ultrareview — no paid plan required.
  Spawns 6 specialized sub-agents simultaneously using the Agent tool, each reviewing from a different angle,
  then synthesizes findings into a ranked, actionable report. Suppresses noise — only HIGH and CRITICAL
  findings reach the report.
  Trigger phrases: "ultrareview", "ultra review", "review this branch", "full review", "pre-commit review",
  "review before PR", "deep review", "run all agents on this", "multi-agent review".
  Args: none = unstaged changes · "staged" = staged only · <PR#> = GitHub PR · <filepath> = single file.
---

# UltraReview — DIY Multi-Agent Code Review

**No paid plan required.** Runs 6 specialized sub-agents in parallel using Claude Code's built-in `Agent` tool.
Each agent reviews from one angle. Findings are merged, deduplicated, and ranked by severity.

---

## Why this works without the paid plan

`/ultrareview` in Claude Code Pro/Max uses cloud infrastructure for parallel agent execution.
This skill does the same thing using the `Agent` tool available in every Claude Code session —
spawning all 6 agents in a single message so they run in parallel. Same result, zero extra cost.

---

## How to use

```
/ultrareview              # review all unstaged changes (git diff HEAD)
/ultrareview staged       # staged changes only
/ultrareview 42           # GitHub PR #42
/ultrareview src/auth.ts  # single file
```

---

## How it works

**You are the orchestrator.** When this skill is invoked:

1. Determine scope (see Scope Detection below)
2. Gather the diff / file content
3. Launch ALL 6 agents in **one message** using parallel Agent tool calls — do not wait for one before starting the next
4. Collect findings from all 6 agents
5. Synthesize into the Final Report

---

## Scope detection

| Invocation | Command |
|------------|---------|
| `/ultrareview` | `git diff HEAD` |
| `/ultrareview staged` | `git diff --cached` |
| `/ultrareview <PR#>` | `gh pr diff <PR#>` |
| `/ultrareview <file>` | Read the file |

If the diff is empty: report "Nothing to review — working tree is clean." and stop.

---

## The 6 agents — launch ALL in one parallel message

Each agent receives the full diff/content block and its role prompt below.
**Every agent must: output ONLY HIGH and CRITICAL findings. Skip style nits, minor suggestions, and subjective preferences.**

---

### Agent 1 — Security
```
You are a security code reviewer. Review the diff for security vulnerabilities only.

Focus:
- Injection: SQL, command, XSS, template injection
- Authentication / authorisation flaws — missing auth checks, broken access control
- Sensitive data exposure — secrets, tokens, PII in code or logs
- Missing input validation at system boundaries
- Insecure direct object references
- Dependency vulnerabilities (flag if you see known-vulnerable package versions)

Output format:
SECURITY | [CRITICAL/HIGH] | <file>:<line> | <one-line description>

CRITICAL = exploitable remotely or exposes user data.
HIGH = exploitable with access or leaks internal state.
Suppress LOW/INFO entirely.
If no HIGH/CRITICAL: output "SECURITY: clean"
```

### Agent 2 — Logic & Bugs
```
You are a logic and bug reviewer. Review the diff for correctness issues only.

Focus:
- Null / undefined dereference before existence check
- Off-by-one errors in loops, slices, date ranges
- Race conditions — async operations on shared state
- Incorrect conditional logic — inverted conditions, unreachable branches
- Missing error handling for operations that can fail
- Data mutation side-effects — functions modifying inputs unexpectedly
- Wrong assumptions about API response shapes

Output format:
LOGIC | [CRITICAL/HIGH] | <file>:<line> | <one-line description>

CRITICAL = data loss, silent corruption, or crash in normal usage.
HIGH = wrong behaviour under predictable conditions.
Skip "could be cleaner" observations — only bugs.
If no HIGH/CRITICAL: output "LOGIC: clean"
```

### Agent 3 — TypeScript Type Safety
```
You are a TypeScript type safety reviewer. Review the diff for type safety issues only.

Focus:
- `as any` or `as unknown as X` casts that bypass type checking
- Non-null assertions (`!`) where null is actually possible
- Missing null/undefined guards before property access
- Type narrowing failures
- Return type mismatches
- Generic type parameters defaulting to `any`
- `@ts-ignore` or `@ts-expect-error` suppressing real errors

Output format:
TYPES | [CRITICAL/HIGH] | <file>:<line> | <one-line description>

CRITICAL = type lie that causes a runtime crash.
HIGH = type lie that produces wrong behaviour silently.
If no HIGH/CRITICAL: output "TYPES: clean"
```

### Agent 4 — Conventions
```
You are a code conventions reviewer. Review the diff against this project's rules.

<!-- ====================================================
  CUSTOMISE THIS BLOCK for your project.
  Replace the rules below with YOUR hard constraints.
  See the "Adapting this skill" section at the bottom.
  ==================================================== -->

Generic rules (replace with project-specific ones):
- No hardcoded credentials, API keys, or secrets
- No relative imports across feature boundaries — use path aliases
- Every database table accessible by end users must have row-level security
- Trial / free-tier limits must match the value defined in config, not hardcoded inline
- No direct DOM manipulation in component files — use framework idioms

Output format:
CONVENTIONS | [CRITICAL/HIGH] | <file>:<line> | <one-line description>

CRITICAL = violates a locked non-negotiable.
HIGH = pattern violation or missing required element.
Skip single-file style nits.
If no HIGH/CRITICAL: output "CONVENTIONS: clean"
```

### Agent 5 — Test Coverage
```
You are a test coverage reviewer. Review the diff for test gaps only.

Focus:
- New functions with no corresponding test
- New API routes / edge functions with no integration test
- Modified behaviour where existing tests no longer cover the new path
- Missing edge case tests for critical logic (auth, payments, domain-critical calculations)
- Test files that assert without actually testing (empty describes, trivial assertions)

Output format:
TESTS | [CRITICAL/HIGH] | <file or function> | <one-line description>

CRITICAL = critical business logic (auth, payment, domain calculations) with no test.
HIGH = new feature path with no test coverage.
Skip test gaps for trivial UI text or purely cosmetic changes.
If no HIGH/CRITICAL: output "TESTS: clean"
```

### Agent 6 — Domain Rules
```
You are the domain-specific rules guardian. Review the diff for violations of this project's
core domain invariants.

<!-- ====================================================
  CUSTOMISE THIS BLOCK.
  Fill in your domain's invariants — the rules where a
  violation is always CRITICAL, not just a style issue.
  Examples for different domains:
  
  Medical/health: "Dosage values must never be negative or zero"
  Finance: "Debit and credit entries must always balance"
  SaaS billing: "Free tier limits are constants — never hardcode a price"
  Insulin app (example): "IOB charts never start at zero — phantom prior dose required"
  
  If you have no domain-specific invariants yet, replace this
  agent with a second Logic pass or a Performance agent.
  ==================================================== -->

Replace this block with your domain invariants before using.

Output format:
DOMAIN | CRITICAL | <file>:<line> | <one-line description>

If no violations: output "DOMAIN: clean"
```

---

## Report format

After collecting all 6 agents' outputs:

```
# UltraReview Report
Scope: <what was reviewed>
Agents: Security · Logic · Types · Conventions · Tests · Domain
Date: <today>

## Critical findings
[All CRITICAL findings, sorted by file]

## High findings
[All HIGH findings, sorted by file]

## Clean passes
[Agents that returned "clean"]

## Summary
<N> critical · <N> high · <N> agents clean
Verdict: [BLOCK MERGE / REVIEW BEFORE MERGE / SAFE TO MERGE]
```

**Verdict rules:**
- Any CRITICAL → **BLOCK MERGE**
- 3+ HIGH → **REVIEW BEFORE MERGE**
- 0-2 HIGH, no CRITICAL → **SAFE TO MERGE**

---

## Noise rules (apply to all agents)

Never include as findings:
- "Consider adding a comment"
- "This could be more readable"
- Missing JSDoc / TSDoc
- Variable name preferences
- Single `console.log` in non-production path
- Formatting (handled by linter)

---

## Adapting this skill

**Agent 4 (Conventions)** — replace the generic rules block with your project's hard constraints.
These are the rules where a violation must be found, not just preferred.

**Agent 6 (Domain)** — this is the most valuable agent to customise. Fill in your domain's
invariants. Examples by domain:

| Domain | Example invariant |
|--------|------------------|
| Medical | Dosage must be positive non-zero float |
| Finance | Debit/credit entries must balance |
| SaaS | Free tier limits come from constants, never hardcoded |
| E-commerce | Inventory can never go negative |
| GluMira | IOB charts never start at zero (phantom prior dose required) |

If you have no domain invariants yet, replace Agent 6 with a **Performance** agent:
- N+1 query patterns
- Unbounded loops over large datasets
- Missing pagination
- Unindexed columns in WHERE clauses

---

## Parallelism note

All 6 agents share the parent context window's budget. Keep the diff focused — very large diffs
(thousands of lines) will exhaust the context. For large PRs, split by directory:
```
/ultrareview src/auth/
/ultrareview src/payments/
```
