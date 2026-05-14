---
name: ruflo
description: >
  Use when the user says "swarm this", "ruflo", "hive-mind", "spawn a swarm",
  "run a council", "10-agent audit", "/ruflo", or when a task is too large or cross-cutting
  for a single Claude session and needs parallel persistent agents with shared memory.
  Ruflo is a third-party MCP server — must be installed before this skill activates.
---

# Ruflo — Swarm / Hive-Mind Orchestration

Ruflo is a multi-agent orchestration MCP server that gives Claude Code a persistent
hive-mind: agents share memory, broadcast to each other, run tasks in parallel,
and leave findings in a shared store that survives session end.

It is NOT the same as spawning background `Agent` calls. Ruflo agents have:
- Persistent identity across sessions (`agent_id`)
- Shared semantic memory (`agentdb_*` tools)
- Direct messaging (`hive-mind_broadcast`, `hive-mind_consensus`)
- Task claiming and work-stealing (prevents double-work across agents)

---

## Install

```bash
# Install Ruflo MCP server (npm package)
npm install -g ruflo-mcp-server

# Or clone and run locally
git clone https://github.com/ruvnet/ruflo
cd ruflo && npm install && npm start
```

Add to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "ruflo": {
      "command": "node",
      "args": ["/path/to/ruflo-mcp-server/index.js"],
      "env": {}
    }
  }
}
```

---

## When to use Ruflo vs standard Agent spawning

| Use standard `Agent` tool | Use Ruflo |
|--------------------------|-----------|
| Short parallel tasks (≤30 min each) | Long-running overnight tasks |
| Task output needed in THIS session | Output survives session end |
| Tasks are independent | Tasks share state and findings |
| One-shot debate (Design Council) | Multi-session audit or build queue |
| ≤5 agents | 5–30 agents |

---

## Core patterns

### 1. Hive-mind init + swarm spawn

```
# Init the hive-mind first
mcp__ruflo__hive-mind_init(swarm_id: "your-swarm-id", config: {...})

# Spawn agents — each gets a role
mcp__ruflo__hive-mind_spawn(swarm_id: "...", agent_role: "auditor", task: "...")
mcp__ruflo__hive-mind_spawn(swarm_id: "...", agent_role: "implementer", task: "...")
mcp__ruflo__hive-mind_spawn(swarm_id: "...", agent_role: "reviewer", task: "...")

# Check status
mcp__ruflo__hive-mind_status(swarm_id: "...")

# Broadcast to all agents
mcp__ruflo__hive-mind_broadcast(swarm_id: "...", message: "Priority changed: focus on auth module")
```

### 2. Shared memory (persist findings across sessions)

```
# Store a finding
mcp__ruflo__agentdb_hierarchical-store(
  key: "audit/auth/finding-001",
  content: "JWT not validated on edge function",
  metadata: { priority: "P0", agent: "security-reviewer" }
)

# Recall all findings by pattern
mcp__ruflo__agentdb_pattern-search(pattern: "audit/auth/*")

# Synthesize across findings
mcp__ruflo__agentdb_context-synthesize(keys: ["audit/auth/*", "audit/data/*"])
```

### 3. Task claiming (prevents double-work)

```
# Create tasks
mcp__ruflo__task_create(title: "Audit auth module", priority: 1)
mcp__ruflo__task_create(title: "Audit payment flow", priority: 2)

# Each agent claims a task before starting
mcp__ruflo__claims_claim(task_id: "...", agent_id: "...")

# Mark complete
mcp__ruflo__task_complete(task_id: "...", result: "...")
```

### 4. Consensus before writing

```
# Agents vote on a decision before it lands in code
mcp__ruflo__hive-mind_consensus(
  swarm_id: "...",
  question: "Should we use JWT or session tokens?",
  options: ["jwt", "session"],
  threshold: 0.7
)
```

---

## `/ruflo` slash command

Install a `/ruflo` Claude Code slash command for one-keystroke swarm launch with built-in Ruflo health check.

Create `~/.claude/commands/ruflo.md` (global — works in every project):

```markdown
---
description: Launch Ruflo 10-agent swarm. Usage: /ruflo <task> — e.g. /ruflo audit auth flow
---

# /ruflo — Swarm

**Ruflo is SELECTED. This command routes directly to the Ruflo MCP — do not respond without calling its tools.**

## Step 1 — Confirm Ruflo is live
Call `mcp__ruflo__system_health` immediately. If any component returns `status: down`, report it and stop.

## Step 2 — Parse the brief
The task brief is: **$ARGUMENTS**
If `$ARGUMENTS` is empty, default brief = `full audit of current working state against all hard constraints in CLAUDE.md`.

## Step 3 — Swarm
Spawn agents via `mcp__ruflo__hive-mind_spawn` or `mcp__ruflo__swarm_init` for the brief above.
```

After saving the file, `/ruflo` appears instantly in Claude Code's command list (no restart needed).

**Usage:**
```
/ruflo                        # full codebase audit
/ruflo audit the auth flow    # focused brief
/ruflo fix IOB chart bug      # targeted swarm
```

---

## GluMira V7 invocation pattern

In the GluMira stack, Ruflo is invoked via the `/ruflo` command or trigger phrase "swarm" / "design council" (10-agent audit variant). The standard council uses 10 agents:

| Agent | Domain |
|-------|--------|
| clinical-reviewer | PK claims, insulin lock, educational disclaimers |
| ux-auditor | Mobile-first, dark/light/auto, no-patient language |
| security-auditor | RLS, input validation, secrets |
| performance-auditor | Build size, render cost, WAAPI compliance |
| content-auditor | Voice standard, Emily filter, source chain |
| accessibility-auditor | a11y, contrast, keyboard nav |
| architect | Module boundaries, data flow |
| test-engineer | Coverage, vitest, e2e |
| devops-engineer | Vercel config, environment variables, CI gates |
| synthesis-lead | Dedupes findings, writes the final report |

---

## Output

Ruflo council output lands in:
- `~/.claude/councils/<date>-<slug>/log.md` — decision log
- Ruflo agentdb — persistent findings queryable in future sessions
- Session summary file — linked at session end

---

## Cost estimate (show before spawning)

| Config | Approx cost |
|--------|------------|
| 5-agent swarm, 30 min | ~15–20k tokens |
| 10-agent full council | ~30–40k tokens |
| Overnight beast mode | ~50–100k tokens |

Always surface the estimate before spawning. User confirms before commit.
