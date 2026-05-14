---
name: openwhispr
description: >
  Voice note capture and transcription via the OpenWhispr CLI. Bridges the
  OpenWhispr desktop app (local, port 8200) or cloud API to Claude Code.
  Use to search, read, create, and manage voice transcriptions directly
  from your Claude session — no context switching.
  Trigger phrases: "openwhispr", "voice notes", "transcription", "whispr notes",
  "search my notes", "get my voice notes".
---

# OpenWhispr — Voice Notes in Claude Code

Bring your OpenWhispr transcriptions into Claude Code. Search recordings,
pull notes into context, create new entries, and manage your voice library
without leaving the terminal.

---

## What OpenWhispr is

OpenWhispr is a voice transcription app that turns recordings into searchable text notes.
The CLI bridges either:

- **Local desktop app** — OpenWhispr running on your machine, port 8200
- **Cloud API** — `api.openwhispr.com` (requires API key)

Claude Code calls the CLI via `Bash`. No MCP server required.

---

## Install

```bash
npm install -g @openwhispr/cli
```

Verify:
```bash
openwhispr --version
openwhispr doctor          # tests connectivity to local + remote backends
```

---

## Setup

### Option A — Local (desktop app)

1. Install the OpenWhispr desktop app from [openwhispr.com](https://openwhispr.com)
2. Open the app — it starts a local bridge on port 8200
3. CLI connects automatically. No auth needed.

```bash
openwhispr --local notes list    # force local backend
```

### Option B — Cloud (API key)

1. Get an API key from [openwhispr.com](https://openwhispr.com)
2. Authenticate:
```bash
openwhispr auth login            # prompts for API key
openwhispr auth status           # confirm it's stored
```

```bash
openwhispr --remote notes list   # force remote backend
```

### Backend selection

By default the CLI auto-selects: local if desktop app is running, otherwise remote.
Override with `--local` or `--remote` flags on any command.

---

## Core commands

### Notes

```bash
# List all notes (20 per page by default)
openwhispr notes list

# List with options
openwhispr notes list --limit 50 --page 2

# Get a single note by ID
openwhispr notes get <id>

# Search notes by keyword
openwhispr notes search "basal test"
openwhispr notes search "Levemir dose"

# Create a new note (text, no audio)
openwhispr notes create --title "Meeting notes" --body "Content here"

# Update a note
openwhispr notes update <id> --title "New title" --body "Updated content"

# Delete a note
openwhispr notes delete <id>
```

### Transcriptions

```bash
# List transcriptions (voice recordings that have been transcribed)
openwhispr transcriptions list

# Get a single transcription
openwhispr transcriptions get <id>

# Delete a transcription (cascades to audio file)
openwhispr transcriptions delete <id>
```

### Audio

```bash
# Delete audio only — keeps the transcribed text
openwhispr audio delete <transcription-id>
```

### Auth & config

```bash
openwhispr auth login            # set API key
openwhispr auth logout           # remove API key
openwhispr auth status           # show auth state

openwhispr config get            # show current config (key redacted)
openwhispr config set <key> <value>   # set a config value
```

### Diagnostics

```bash
openwhispr doctor                # test local + remote connectivity
openwhispr doctor --local        # test local bridge only
openwhispr doctor --remote       # test cloud API only
```

---

## Claude Code usage patterns

### Pull recent notes into context

Ask Claude:
> "Get my last 10 voice notes and show me what I recorded this week."

Claude calls:
```bash
openwhispr notes list --limit 10
```

Then reads the output and summarizes.

### Search for a specific topic

> "Search my openwhispr notes for anything about basal testing."

```bash
openwhispr notes search "basal testing"
```

### Capture a thought as a note

> "Create a note titled 'V7 launch blocker' with the content: diet modules must ship at v1."

```bash
openwhispr notes create --title "V7 launch blocker" --body "diet modules must ship at v1"
```

### Bring a transcription into the working session

> "Get the transcription from my Monday morning recording — ID abc123."

```bash
openwhispr transcriptions get abc123
```

Paste the output into context for Claude to summarize, extract action items, or convert to a spec.

### Clean up old audio (keep text)

> "I've confirmed the notes are accurate. Delete the audio file for transcription xyz to save space."

```bash
openwhispr audio delete xyz
```

---

## Adding to CLAUDE.md

```markdown
## Voice notes
OpenWhispr CLI installed globally. Use `openwhispr notes search <query>` to
find voice recordings. Default backend: local if desktop app running, else cloud.
API key stored via `openwhispr auth login`.
```

---

## Backend decision

| Situation | Use |
|-----------|-----|
| Desktop app open and running | `--local` (zero latency, no quota) |
| On another machine / no desktop app | `--remote` (needs API key) |
| Unsure | Omit flag — CLI auto-detects |
| Debugging connectivity | `openwhispr doctor` first |

---

## Why this matters for Claude Code sessions

Voice notes are where thinking happens outside the IDE — during walks, before sleep,
in the car. OpenWhispr captures that thinking. This integration means:

- Brainstorming recorded at 11pm → pulled into the next morning's session as context
- Architecture decisions recorded verbally → formatted as a decision log entry
- Bug descriptions recorded on the go → turned into KNOWN_ISSUES.md entries

The friction is the copy-paste from app to IDE. This skill removes it.
