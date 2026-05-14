---
name: content-os
description: >
  Use when writing any content — social posts, product copy, clinical/technical content,
  personal stories, or auditing existing content before publish.
  Trigger phrases: "write a post", "content for X", "draft copy", "audit this before I post",
  "check this content", "brand voice check", "content OS".
  Loads brand voice, content type rules, source chain, profile variables, and quality filter.
---

# Content OS

A structured operating system for Claude-generated content. Prevents the four failure modes
that make AI-written content visibly AI: vague claims, wrong tone, unsourced facts, and
one-size-fits-all copy that ignores the audience.

Install the five files in `.claude/content-os/`. Claude reads them at content-creation time.

---

## The five files

### 1. BRAND-VOICE.md

Your single source of truth for voice. The most important file.

**Template:**

```markdown
# [Your Brand] — Brand Voice

## Core test
Every sentence must pass: **Is this claim falsifiable?**
If yes — write it. If no — rewrite until it is, or don't write it.

## What this kills
| Write this | Not this |
|---|---|
| [Specific, verifiable claim] | [Vague feel-good statement] |
| [Data point] | [Adjective-heavy assertion] |

## Voice by surface
| Surface | Tone | Length | Emoji |
|---|---|---|---|
| Social (community) | Warm, direct | 150–300 words | Minimal |
| LinkedIn | Professional, evidence-forward | 200–400 words | None |
| Product copy | Direct, second person | As short as possible | None |
| Technical/clinical | Precise, cited | As long as the claim needs | None |

## What [your brand] never says
- [banned phrase 1]
- [banned phrase 2]
- Anything sourced only from LLM training data
```

---

### 2. CONTENT-TYPES.md

Rules and templates per content type. One section per type — Claude reads only the section it needs.

**Template:**

```markdown
# Content Types

## § Social — Facebook / LinkedIn / Instagram

### Facebook (community)
Structure:
1. Hook — a specific, true detail (not a question, not a generic opener)
2. The reality — what actually happened or what is true
3. The relevance — why this matters to your audience
4. Optional CTA — never salesy

Rules:
- Every claim passes SOURCE-CHAIN check before posting
- Quality filter mandatory
- No invented details

Multi-format flag: one post → LinkedIn long-form → Instagram carousel → in-product tip → email

## § Product copy
Rules:
- Second person always ("you", "your")
- Never jargon the audience doesn't already know
- Educational disclaimer on all analytical surfaces

## § Story / personal content
Rules:
- Write ONLY what the source provides
- No invented atmosphere, inferred emotion, assumed timeline
- Missing detail = [CONFIRM: X] — not a gap-fill
```

---

### 3. SOURCE-CHAIN.md

Where every claim must originate. Prevents hallucinated facts.

**Template:**

```markdown
# Source Chain

Every claim in [your brand] content must trace to a source in this chain.
Tier 1 = highest authority. Never cite a lower tier when a higher one exists.

## Tier 1 — Your own product/codebase (highest authority)
If the code says X, content says X.
[List your source files and what each is authoritative for]

## Tier 2 — Cited peer-reviewed research
Named citations with PMID/DOI. Never paraphrase a citation you haven't read.

## Tier 3 — Named standards / guidelines
[Your field's official guidelines — specify year]

## Tier 4 — Founder/expert-provided primary knowledge
For personal, story, or lived-experience content only.
Everything must be explicitly provided by the human — nothing inferred.

## Tier 5 — General knowledge / LLM training data (LOWEST — use sparingly)
Flag explicitly when used: [TIER 5 — verify before publishing]
```

---

### 4. PROFILE-VARIABLES.md

Variables that change the content. Prevents one-size-fits-all copy.

**Template:**

```markdown
# Profile Variables

Before writing any personalised content, confirm these variables.
A general assumption is wrong. Every variable listed here changes the output.

## Step 0 — Ask for context if not provided

Variables to confirm:
1. AUDIENCE: [who is this for?]
2. CONTEXT: [what situation are they in?]
3. [Domain-specific variable 1]
4. [Domain-specific variable 2]

If writing general educational content: state that explicitly and apply
the broadest safe language. Acknowledge the variable range, don't pick one.

## Variable reference
[Your domain-specific variable definitions]
```

---

### 5. QUALITY-FILTER.md (the Emily Filter, generalised)

Named for the real person who will publicly correct you if you get it wrong.
Every domain has one — the expert with lived experience whose corrections are already on record.

**Template:**

```markdown
# Quality Filter

Named for [the person in your domain who will correct you publicly].
This filter exists so they never have to.

## Who [name] is as a validator
[Name] is not a hypothetical reviewer. They are:
- [Specific lived-experience credential 1]
- [Specific lived-experience credential 2]
- [Known correction on record]

## Known corrections (LOCKED — these are wrong, never repeat them)
| Wrong claim | Why wrong | What to write instead |
|---|---|---|

## How to run the filter

Q1 — The public correction test
"If [name] saw this published, would they write a correction?"
- If yes: rewrite. If uncertain: flag.

Q2 — The reality test
"Does this hold true in [your specific context], not in the generic case?"

Q3 — The completeness test
"Does this acknowledge the full weight of the topic without dramatising or minimising it?"

Q4 — The variable test
"Would this claim be wrong for [edge case in your domain]?"

Q5 — The invention test
"Did any detail come from imagination rather than a verified source?"

## Filter result format
Every piece of content must include:
QUALITY FILTER: passed | flagged — [specific issue]
If flagged: do not output. Fix first.
```

---

## How to install

1. Copy the five files into `.claude/content-os/` in your project
2. Fill in your brand specifics in each template
3. Add to your CLAUDE.md or boot command: "Read `.claude/content-os/` before writing any content."
4. Optionally wire into a `/content` slash command that loads all five files before writing

---

## The pre-publish checklist

Before any piece of content ships, run in this order:

1. **Invention check** — every detail traceable to a source?
2. **Source chain** — every claim at the correct tier?
3. **Quality filter** — all 5 questions answered?
4. **Variable check** — anything stated as universal that only applies in one context?
5. **Voice check** — does every sentence pass the falsifiability test?
6. **Audience routing** — is this voiced for the right surface?

All 6 pass = clear to publish.
