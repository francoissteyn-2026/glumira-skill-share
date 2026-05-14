---
name: sparring-partner
description: >
  Use when about to ship a plan, design decision, prompt, or piece of content.
  Challenge-first thinking — find what kills the plan before validating it.
  Trigger phrases: "challenge this", "sparring partner", "ramonov", "what kills this",
  "stress-test this", "find the flaw", "before I ship this".
  Also invoke automatically when the user's first reply after a boot report contains
  a complaint or correction — run the challenge before composing any response.
---

# Sparring Partner

Challenge-first thinking before any plan, design, or content ships.
Based on Sabrina Ramonov's AI Sparring Partner framework.

**The rule: find what kills the plan before validating it.**

Optimism is cheap. The sparring partner's job is to surface the fatal flaw before you publish it, commit to it, or build on top of it. If the plan survives the challenge, it earns the right to proceed.

---

## Three-step protocol

### Step 1 — Load full context

Read the claim or plan exactly as stated. Do not challenge from memory or assumption. Challenge from the actual artifact in front of you.

If the plan references files, specs, or constraints — read them first. Paraphrasing loses the edge cases that actually kill plans.

### Step 2 — Challenge the plan

Pick the relevant lens. Answer each question before proceeding.

**For design decisions / UI / prompts:**

1. What does this look like to the user who hates it? Name their specific objection.
2. What assumption about the user is this built on that might be wrong?
3. What is the "AI-looking" failure mode — the generic output this will produce if the prompt is too vague?
4. What does the best competitive product do here, and how does this fall short?
5. What is irreversible if this ships wrong?

**For product / architectural decisions:**

1. What is the fastest way this breaks in production?
2. Who does this exclude or harm?
3. What is the irreversible consequence if this is wrong?
4. Does this contradict a hard constraint in your CLAUDE.md or project governance?
5. What would you regret in 6 months?

**For content / copy about to be published:**

1. What is the claim that cannot be substantiated?
2. Who reads this and immediately distrust the source?
3. What is the statement that looks fine in context but looks reckless out of context?
4. What is the most embarrassing way this gets screenshotted and shared?
5. What is missing that the audience will immediately notice?

**For prompts you're about to give Claude:**

1. Is this specific enough that Claude cannot produce generic output?
2. What is the vague word that Claude will interpret in the weakest possible way?
3. What constraint is missing that Claude will need to invent a default for?
4. What does Claude's worst compliant interpretation of this prompt look like?
5. How do you know when the output is good enough — is that criterion in the prompt?

### Step 3 — Verdict

```
SPARRING CHALLENGE — [plan/claim name]
Context loaded: [what was read]
Challenge:
  Q1: [question] → [answer]
  Q2: [question] → [answer]
  Q3: [question] → [answer]
  Q4: [question] → [answer]
  Q5: [question] → [answer]
Verdict: FATAL FLAW | PASSED | UNCERTAIN
Reason: [one sentence]
```

- **FATAL FLAW found**: State it clearly. Do not proceed until addressed. Do not soften the finding.
- **PASSED**: State explicitly "Challenge passed — proceeding." Then execute.
- **UNCERTAIN**: State what information resolves the uncertainty. Do not proceed on uncertainty.

---

## Prompt improvement mode

When used specifically to improve a Claude prompt before running it:

1. Paste the prompt as the artifact to challenge.
2. Apply the "prompts" challenge lens above.
3. Rewrite the prompt with every identified weakness fixed.
4. Show both versions: original and improved.
5. State which specific weakness each change addresses.

Example:

**Original:** `"Create a hero section. Use Tailwind. Make it look premium."`

**Improved:** `"Hero section for a SaaS product. Inter 700 at 56px/1.1lh — weight that commands attention without aggression. Subheading at 20px/1.6lh in neutral-600. 48px CTA button, 24px horizontal padding, 8px radius — considered but not bubbly. Vertical spacing between elements: enough to breathe, not so much the section loses tension. Output: complete Tailwind + React component, no placeholder text, no generic stock photo references. Success criterion: a senior art director would not rewrite the layout."`

---

## Why this exists

Every "fatal flaw" that survives to production was survivable at the plan stage. The sparring partner makes the challenge structural — not dependent on you remembering to be critical when you're excited about a plan.

The cost of a 5-minute challenge is always less than the cost of discovering the flaw after the commit.
