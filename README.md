# GluMira Skill Share

Claude Code skills extracted from the GluMira V7 production stack and generalised for public use.

Run (skills/adversarial-audit/) against your stack to see if it would be beneficial to your workflow.

These are not templates. They are working skills battle-tested on a real SaaS product built solo with Claude Code — a clinical insulin-tracking platform for T1D families.

## Skills

| Skill | What it does | Install time |
|-------|-------------|--------------|
| [`adversarial-audit`](skills/adversarial-audit/) | Attack any framework/skill/stack before adopting it. 5 angles, per-component verdict. | 1 min |
| [`project-setup`](skills/project-setup/) | Step-by-step start + scope for any new project. 15 min → saves 15 sessions. | 15 min |
| [`sparring-partner`](skills/sparring-partner/) | Challenge-first thinking. Find what kills the plan before you build it. | 1 min |
| [`design-council`](skills/design-council/) | 9–11 parallel specialist agents debate your design in real time | 1 min |
| [`animejs`](skills/animejs/) | Anime.js v4 ESM API + React patterns for premium motion | 1 min |
| [`higgsfield`](skills/higgsfield/) | Cinematic AI video + talking-head generation via Higgsfield MCP | 10 min |
| [`superpowers`](skills/superpowers/) | 5-agent parallel execution — compress timeline, not quality | 1 min |
| [`ruflo`](skills/ruflo/) | Swarm orchestration — persistent hive-mind for multi-session tasks | 20 min |
| [`content-os`](skills/content-os/) | Brand voice + content types + source chain + quality filter system | 20 min |
| [`obsidian`](skills/obsidian/) | Obsidian vault integration — wikilinks, memory, canonical structure | 5 min |
| [`compact`](skills/compact/) | 15–20 message session cadence — kills context bloat | 1 min |
| [`closing-protocol`](skills/closing-protocol/) | Three-option task closer — locked, no variations | 1 min |
| [`get-shit-done`](skills/get-shit-done/) | Behavioral override — execution mode, no permission-seeking | 1 min |
| [`state-tracker`](skills/state-tracker/) | STATE.md + KNOWN_ISSUES.md — context continuity across sessions | 5 min |
| [`openwhispr`](skills/openwhispr/) | Voice notes → Claude context. Search + pull transcriptions without leaving the terminal. | 2 min |
| [`hermes`](skills/hermes/) | NousResearch Hermes Agent as MCP server — web search, cron, Telegram gateway, subagents | 15 min |
| [`playwright`](skills/playwright/) | Microsoft Playwright MCP — full browser automation, screenshots, E2E flows, post-deploy checks | 2 min |

## Install

Copy the skill folder into your `.claude/skills/` directory:

```
your-project/
└── .claude/
    └── skills/
        └── sparring-partner/    ← paste folder here
            └── skill.md
```

Claude Code auto-discovers skills in `.claude/skills/`. Invoke with the Skill tool or by saying the trigger phrase listed in each skill's frontmatter.

## The order that works for a new project

```
1. project-setup     → governance, memory, state tracking, hooks — do this FIRST
2. sparring-partner  → challenge your scope before building anything
3. state-tracker     → wires STATE.md + KNOWN_ISSUES.md into your boot sequence
4. obsidian          → connects your repo to Obsidian vault (optional but powerful)
5. design-council    → for every significant design decision, before writing code
6. animejs           → premium motion on top of the polished layout
7. higgsfield        → cinematic video when you need it
8. superpowers       → parallel execution when a task needs depth across 5 domains
9. ruflo             → swarm for overnight or multi-session tasks
10. content-os       → structured content system when you're writing for an audience
11. compact          → session discipline — fresh every 15-20 messages
12. closing-protocol → LOCKED — every task ends with 1/2/3, always
13. get-shit-done    → behavioral override — ship, don't ask
14. openwhispr       → pull voice notes into session context (optional, needs desktop app or API key)
15. hermes           → standalone agent as MCP server — Telegram gateway, cron, web search, subagents
16. playwright       → browser automation — verify every deploy before you summarize and commit
```

## Design council — full protocol included

The `design-council` skill ships with the complete protocol reference:
- `references/protocol.md` — full 6-phase orchestration guide
- `references/opening-prompt-template.md` — spawn prompt scaffold
- `references/decision-log-template.md` — log format
- `references/implementation-handoff.md` — how to ship the decision
- `references/review-mode.md` — codebase audit variant
- `references/roles/` — 15 role briefs (principal-engineer through ui-ux-designer)

## Stack this runs on

- Claude Code (CLI) with hooks + skills
- Vite 6, React 19, TypeScript 5.6
- Tailwind v4, shadcn/ui
- Supabase, Vercel
- Obsidian (vault = repo root)
- Ruflo MCP, Higgsfield MCP

## License

MIT. Use, adapt, share. Attribution appreciated but not required.

---

Built by [@francoissteyn01](https://github.com/francoissteyn01-hash) — GluMira V7
