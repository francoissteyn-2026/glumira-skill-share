---
name: higgsfield
description: >
  Use when generating cinematic AI video, talking-head video, image-to-video,
  or AI images via Higgsfield. Trigger phrases: "generate a video", "make a video",
  "talking head", "cinematic shot", "higgsfield", "animate this image".
  Requires Higgsfield API key and the higgsfield MCP server configured.
---

# Higgsfield MCP — Cinematic AI Video & Image Generation

Higgsfield AI specialises in cinematic-quality video generation — motion, talking heads,
image-to-video, and style-consistent sequences. Accessed via their remote MCP server.

---

## Setup (5 minutes)

### Step 1 — Get API credentials

1. Go to [higgsfield.ai](https://higgsfield.ai) and create an account
2. Navigate to **Settings → API Keys**
3. Copy your API key and secret

### Step 2 — Add to `.mcp.json` in your project root

```json
{
  "mcpServers": {
    "higgsfield": {
      "type": "http",
      "url": "https://mcp.higgsfield.ai/mcp",
      "headers": {
        "Authorization": "Bearer ${HIGGSFIELD_API_KEY}",
        "X-Secret": "${HIGGSFIELD_SECRET}"
      }
    }
  }
}
```

### Step 3 — Set environment variables

**Windows (PowerShell):**
```powershell
$env:HIGGSFIELD_API_KEY = "your-api-key-here"
$env:HIGGSFIELD_SECRET = "your-secret-here"
```

**Mac/Linux:**
```bash
export HIGGSFIELD_API_KEY="your-api-key-here"
export HIGGSFIELD_SECRET="your-secret-here"
```

**Persistent (add to shell profile or `.env`):**
```bash
# .env (if using dotenv)
HIGGSFIELD_API_KEY=your-api-key
HIGGSFIELD_SECRET=your-secret
```

### Step 4 — Enable in Claude Code settings

In `.claude/settings.json`:
```json
{
  "enabledMcpjsonServers": ["higgsfield"]
}
```

### Step 5 — Verify

Start a new Claude Code session. In the session:
```
/mcp
```
You should see `higgsfield` in the connected servers list.

---

## Core capabilities

| Tool | What it does |
|------|-------------|
| `generate_video` | Text-to-video, image-to-video, style transfer |
| `generate_image` | Cinematic image generation |
| `job_display` | Check generation job status and retrieve output |
| `show_generations` | List your recent generations |
| `media_upload` | Upload source image for image-to-video |
| `balance` | Check your Higgsfield credit balance |
| `show_characters` | Manage talking-head character profiles |
| `virality_predictor` | Predict viral potential of content |

---

## Usage patterns

### Generate a cinematic video

```
Trigger: "generate a cinematic video of [description]"

Claude will call:
mcp__higgsfield__generate_video({
  prompt: "...",
  style: "cinematic",
  duration: 5
})
```

### Talking-head video from script

```
mcp__higgsfield__show_characters()  → pick character ID
mcp__higgsfield__generate_video({
  character_id: "...",
  script: "Your spoken text here",
  background: "professional studio"
})
```

### Image to video

```
1. mcp__higgsfield__media_upload({ file_path: "path/to/image.jpg" })
2. mcp__higgsfield__generate_video({
     source_image_url: "<url from upload>",
     motion_prompt: "slow zoom in, cinematic"
   })
```

### Check job status

```
mcp__higgsfield__job_display({ job_id: "..." })
```

---

## Prompt patterns for premium output

Generic: `"A person walking in a city"`

Premium: `"Golden hour. A lone figure walks through a fog-drenched street in Osaka. Anamorphic lens flare. Film grain. 24fps. Wim Wenders colour grade. Slow motion. No camera shake."`

Key elements for cinematic quality:
- **Lighting:** golden hour, blue hour, neon, fog, dusk
- **Lens language:** anamorphic, 50mm, shallow depth of field
- **Film language:** colour grade reference (Wenders, Tarkovsky, Wong Kar-wai)
- **Movement:** slow zoom, dolly in, static wide, handheld
- **Technical:** 24fps, film grain, 4K

---

## Credit costs

Generation costs vary by duration and resolution. Check balance before long sessions:
```
mcp__higgsfield__balance()
```

Estimate before bulk generation:
```
mcp__higgsfield__show_plans_and_credits()
```
