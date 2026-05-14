---
name: hermes
description: >
  Hermes Agent by NousResearch — standalone AI agent platform that runs as an
  MCP server Claude Code can consume. Adds web search, file ops, cron jobs,
  Telegram/Discord gateway, session recall, and multi-model delegation to your
  Claude Code stack. Install once, wire as MCP, get Hermes tools in every session.
  Trigger phrases: "hermes", "install hermes", "hermes agent", "wire hermes",
  "hermes mcp".
---

# Hermes Agent — MCP Server for Claude Code

Hermes Agent by NousResearch is a full AI agent platform. When wired as an MCP
server, Claude Code gains access to Hermes's entire tool suite: web search,
file operations, session history, cron scheduling, Telegram/Discord gateway,
and multi-model delegation — without leaving the Claude Code session.

**The pattern:** Hermes runs as a subprocess MCP server. Claude Code discovers
it at boot and calls its tools like any other MCP tool.

---

## What Hermes adds to your stack

| Tool | What it does |
|------|-------------|
| `web_search` | AI-native web search |
| `web_extract` | Full page content extraction |
| `terminal` | Command execution (local or remote) |
| `read_file` / `write_file` | File operations |
| `browser_*` | Full browser automation |
| `cronjob` | Schedule recurring agent tasks |
| `session_search` | Search past Hermes conversations |
| `delegate_task` | Spawn parallel child agents |
| `skills_list` / `skill_view` | Browse loaded skills |
| `image_generate` | Image generation (FAL.ai) |
| `text_to_speech` | TTS (Edge TTS free) |

---

## Install

> ### ⚠ Windows: install from YOUR OWN terminal, NOT from inside Claude Code
>
> Claude Code on Windows is a packaged (MSIX) app with identity `Claude_pzs8sxrjxfjjc`.
> Anything Claude's Bash/PowerShell tools install at `C:\Users\<you>\AppData\Local\...`
> gets silently **virtualized** to `C:\Users\<you>\AppData\Local\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Local\...`.
> Inside Claude, both paths look identical (`Test-Path` returns true, `where hermes` finds it).
> **Outside Claude, that path is empty.** Worse — the generated `hermes.exe` launcher
> embeds a hardcoded virtualized path for `python.exe`, so the binary cannot run
> from your terminal even with the full Packages\ path.
>
> **Symptom:** you `irm | iex` the installer through Claude, everything looks fine,
> but in your own PowerShell `hermes` returns "not recognized as the name of a cmdlet"
> — and pasting the full path returns the SAME error.
>
> **Fix:** open YOUR PowerShell (Start Menu → Windows PowerShell), then run the
> installer line below. Do not run it through Claude Code's terminal.

### Windows (PowerShell)

```powershell
irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex
```

### macOS / Linux (bash)

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

The installer handles Python, uv, Node.js, ripgrep, ffmpeg, and Playwright.
On Windows, `pywinpty` requires MSVC — if not present, the installer falls
through to "core only" (all tools except the TUI). Core is sufficient for MCP.

**Important — venv naming:** the Windows installer creates `hermes-agent\venv\`
(no leading dot), not `.venv\`. Update every PATH and `.mcp.json` reference
accordingly. The older skill version used `.venv` — that was from a previous
installer release and is wrong for the current script.

**Verify:**
```bash
hermes --version
hermes doctor
```

---

## Configure API key

Hermes supports OpenRouter, Anthropic direct, Gemini, OpenAI, and others.
For a Claude Code stack where you already have an Anthropic key:

**macOS / Linux:**
```bash
echo "ANTHROPIC_API_KEY=sk-ant-..." >> ~/.hermes/.env
```

**Windows (PowerShell):**
```powershell
Add-Content "$env:LOCALAPPDATA\hermes\.env" "`nANTHROPIC_API_KEY=sk-ant-..."
```

Then set the provider in `~/.hermes/config.yaml` (or `%LOCALAPPDATA%\hermes\config.yaml` on Windows):

```yaml
model:
  default: "claude-sonnet-4-6"
  provider: "anthropic"
```

**Note on the setup wizard's default model:** the installer's setup wizard may
write an older Sonnet ID (e.g. `claude-sonnet-4-5-20250929`) as the default,
even when you select Anthropic. Switch to the latest inside the TUI:

```
/model claude-sonnet-4-6 --global
```

The `--global` flag persists the choice to `config.yaml`. Without it the
switch is session-only and the next `hermes` launch reverts. You may see
"could not verify `claude-sonnet-4-6` against this endpoint's model listing"
— this is benign; Anthropic doesn't expose `GET /v1/models` so hermes can't
enumerate. The model is accepted and billed normally.

---

## Wire as MCP server

### Step 1 — Add to `.mcp.json` (project root)

**macOS / Linux:**
```json
{
  "mcpServers": {
    "hermes": {
      "command": "hermes",
      "args": ["mcp", "serve", "--accept-hooks"],
      "env": {
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

**Windows (full path required):**
```json
{
  "mcpServers": {
    "hermes": {
      "command": "C:\\Users\\<you>\\AppData\\Local\\hermes\\hermes-agent\\venv\\Scripts\\hermes.exe",
      "args": ["mcp", "serve", "--accept-hooks"],
      "env": {
        "HERMES_HOME": "C:\\Users\\<you>\\AppData\\Local\\hermes",
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

Replace `<you>` with your Windows username. Note: `venv` not `.venv` — the
current installer drops the leading dot.

### Step 2 — Activate in `.claude/settings.json`

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}"
  },
  "enabledMcpjsonServers": ["hermes"]
}
```

### Step 3 — Verify

In a Claude Code session, run `/mcp` — you should see `hermes` listed with its tools.

---

## Link your skill library (optional)

Point Hermes at your skill folder so it inherits the same skills as Claude Code:

In `~/.hermes/config.yaml`:

```yaml
skills:
  external_dirs:
    - /path/to/your/skills
```

Hermes loads these as read-only. Skills you create via `hermes` go to `~/.hermes/skills/`.

---

## Usage patterns

### Web search from Claude Code

Claude Code calls Hermes's `web_search` tool directly — no prompt needed.
Or explicitly: "Use Hermes to search for [topic]."

### Schedule a recurring task

```
Ask Hermes to run `git pull && npm run build` every day at 6am.
```

Hermes creates a cron job. Manage with `hermes cron list`.

### Recall a past session

```
Search Hermes sessions for the conversation about basal testing from last week.
```

Hermes uses full-text + semantic search across its session logs.

### Spawn a parallel subagent

```
Delegate to Hermes: research the top 3 competitors and return a comparison table.
```

Hermes runs a child agent in isolated context, returns the result.

### Telegram / Discord gateway

```bash
hermes gateway start
```

Hermes runs as a background agent you can message from Telegram or Discord.
Useful for overnight tasks — message it, get notified when done.

---

## Hermes vs Ruflo

Both are swarm/agent tools. They solve different problems:

| | Ruflo | Hermes |
|---|---|---|
| Lives in | Claude Code native MCP | Standalone agent, exposed as MCP |
| Memory | Semantic vector store, persistent | Per-session + MEMORY.md + USER.md |
| Best for | Multi-agent coordination inside Claude | Gateway (Telegram/Discord), cron, standalone runs |
| Model | Any (routed by Ruflo) | Any (direct API or OpenRouter) |
| Skills | No | Yes — same format as Claude Code |

**The stack:** Ruflo for Claude-native swarms, Hermes for standalone gateway tasks
and when you need a second agent that persists between Claude sessions.

---

## First run check

```bash
hermes chat
```

Should open the TUI with your model loaded. Type `hello` to confirm connectivity.
If it fails, run `hermes doctor` — it diagnoses provider auth, PATH, and config.

---

## Troubleshooting

**Windows: `hermes` not found after install (you ran the installer from your OWN terminal)**
The installer adds `venv\Scripts` to **User** PATH. If you opened a terminal
*before* the installer finished, restart that terminal. Fresh terminals will
pick up the new PATH automatically.

If you need hermes available in **admin/elevated** terminals too, add it to
Machine PATH (one-time, elevated):
```powershell
$scripts = "$env:LOCALAPPDATA\hermes\hermes-agent\venv\Scripts"
$mp = [Environment]::GetEnvironmentVariable("PATH","Machine")
if ($mp -notlike "*$scripts*") {
    [Environment]::SetEnvironmentVariable("PATH", "$mp;$scripts", "Machine")
}
```

**Windows: `hermes` not found after install (you ran the installer from inside Claude Code)**
See the warning at the top of "Install". Symptom: the binary appears to exist
inside Claude but doesn't exist in your own terminal. Fix: re-run the installer
from your OWN PowerShell. The Claude-side install is trapped in the MSIX
sandbox and unrecoverable from outside.

**`hermes_cli` ModuleNotFoundError on setup wizard**
Harmless — occurs because the setup wizard launches before the venv is on PATH.
The install itself succeeded. Run `hermes chat` directly.

**`pywinpty` build failure (Windows)**
Requires MSVC Build Tools. Either install them or use core-only (sufficient for MCP).
Core-only is what the installer falls back to automatically.
