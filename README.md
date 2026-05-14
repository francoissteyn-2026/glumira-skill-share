# GluMira Skill Share

Claude Code skills extracted from the GluMira V7 production stack and generalised for public use.

These are not templates. They are working skills battle-tested on a real SaaS product (GluMira — a clinical insulin-tracking platform built solo with Claude Code).

## Skills

| Skill | What it does | Best for |
|-------|-------------|----------|
| [`sparring-partner`](skills/sparring-partner/) | Challenge-first thinking before any plan ships | Eliminating fatal flaws before you build |
| [`design-council`](skills/design-council/) | Parallel multi-agent design debate (9–11 specialists) | Premium UI decisions, landing pages, dashboards |
| [`animejs`](skills/animejs/) | Anime.js v4 ESM API + React patterns | Premium motion and micro-interactions |
| [`get-shit-done`](skills/get-shit-done/) | Behavioral override — execution mode, no permission-seeking | Killing Claude's default cautious posture |
| [`state-tracker`](skills/state-tracker/) | STATE.md + KNOWN_ISSUES.md scaffold | Context continuity across sessions |

## Install

Each skill is a single folder. Copy the skill folder into your `.claude/skills/` directory:

```
your-project/
└── .claude/
    └── skills/
        └── sparring-partner/    ← paste folder here
            └── skill.md
```

Claude Code auto-discovers skills in `.claude/skills/`. Invoke with the Skill tool or by saying the trigger phrase listed in each skill's description.

## The order that works

1. **Sparring partner first** — before you write any brief, run it through the sparring partner. Kill the fatal flaw before it costs you tokens.
2. **Design council** — once the direction survives the challenge, convene parallel specialists to debate the actual design. Structural disagreement, not one AI opinion.
3. **Anime.js** — motion is 50% of the gap between "AI-looking" and premium.
4. **GSD** — behavioral override that stops Claude asking for permission on every step.

## Stack this runs on

- Claude Code (CLI)
- Vite 6, React 19, TypeScript 5.6
- Tailwind v4, shadcn/ui
- Supabase, Vercel

## License

MIT. Use, adapt, share.
