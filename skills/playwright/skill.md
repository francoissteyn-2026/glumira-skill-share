---
name: playwright
description: >
  Microsoft Playwright MCP server — gives Claude Code full browser automation.
  Navigate, click, fill forms, take screenshots, capture console logs, monitor
  network requests, and verify UI without leaving the terminal. The correct
  tool for post-deploy visual checks, E2E flows, and scraping tasks.
  Trigger phrases: "playwright", "browser automation", "screenshot", "test UI",
  "verify the page", "check the deploy", "e2e", "browser test".
---

# Playwright MCP — Browser Automation for Claude Code

The Microsoft Playwright MCP server wires a full Chromium browser into
Claude Code. Every navigation, click, form fill, and screenshot is a tool
call. No switching to a browser manually, no copy-pasting URLs, no "can you
check if it looks right" — Claude verifies it directly.

**The rule:** if you've just shipped a UI change, Claude verifies it in the
browser before the session closes. No exceptions.

---

## Install

No global install required — run on demand via `npx`:

```bash
npx @playwright/mcp@latest --help
```

Or install globally for faster startup:

```bash
npm install -g @playwright/mcp
```

Playwright downloads Chromium automatically on first run (~150MB).

---

## Wire as MCP server

### `.mcp.json` (project root)

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

**Headed mode** (see the browser window — useful for debugging):
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--headed"]
    }
  }
}
```

**Specific viewport** (mobile simulation):
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--viewport-size", "390,844"
      ]
    }
  }
}
```

### Activate in `.claude/settings.json`

```json
{
  "enabledMcpjsonServers": ["playwright"]
}
```

### Verify

In a Claude Code session: `/mcp` → playwright should appear with its tools listed.

---

## Core tools

| Tool | What it does |
|------|-------------|
| `browser_navigate` | Go to a URL |
| `browser_snapshot` | Capture accessibility tree (for structure/content checks) |
| `browser_take_screenshot` | PNG screenshot of current page |
| `browser_click` | Click any element |
| `browser_fill` | Type into an input |
| `browser_select_option` | Choose from a dropdown |
| `browser_press_key` | Keyboard shortcut (Enter, Escape, Tab, etc.) |
| `browser_hover` | Hover over an element |
| `browser_drag` | Drag-and-drop |
| `browser_evaluate` | Run JavaScript in the page context |
| `browser_console_messages` | Read browser console output |
| `browser_network_requests` | Inspect network calls |
| `browser_wait_for` | Wait for element, URL, or network idle |
| `browser_tabs` | Manage multiple tabs |
| `browser_close` | Close browser |

---

## Usage patterns

### Post-deploy verification

After every Vercel deploy, Claude navigates to the preview URL and checks:

```
Navigate to [preview URL].
Screenshot the landing page.
Check the console for errors.
Click the primary CTA and confirm the next page loads correctly.
```

No more "it looked fine locally."

### Visual regression check

```
Open [local dev URL] at 1440px wide. Screenshot the dashboard.
Now resize to 375px. Screenshot again.
Check for: text overflow, broken grid, hidden buttons, overlapping elements.
```

### Form flow testing

```
Navigate to [sign-up URL].
Fill: email = test@example.com, password = Test1234!
Click Submit.
Wait for navigation.
Screenshot the result — confirm success state, not error state.
```

### Scraping structured data

```
Navigate to [URL].
Extract all pricing tier names and monthly prices from the page.
Return as a JSON array.
```

### Console error audit

```
Navigate to [URL].
Capture all console errors and warnings.
List any 4xx/5xx network requests.
```

### Authentication flow

```
Navigate to [login URL].
Fill credentials.
Submit.
Confirm redirect to dashboard.
Screenshot the authenticated state.
```

---

## Integrating with the closing protocol

Add browser verification to your task closer. Before running `2. Summarize / Save`:

```
1. Navigate to dev URL
2. Screenshot golden paths (landing, main feature, auth)
3. Check console — no new errors
4. THEN summarize and commit
```

This makes visual verification part of the session discipline, not an afterthought.

---

## Integrating with design-council

After a design-council decision ships:

```
Navigate to [URL].
Screenshot the component that was debated.
Compare against the council's decision log.
Flag any divergence between decision and implementation.
```

The council decides. Playwright confirms the decision landed correctly.

---

## Headed vs headless

| Mode | Use when |
|------|---------|
| Headless (default) | CI, automated checks, scraping — fast, no window |
| `--headed` | Debugging a failing flow — see exactly what Claude sees |

Switch in the args: `"--headed"` for debugging, remove it for production.

---

## Troubleshooting

**Playwright can't find Chromium**
```bash
npx playwright install chromium
```

**Timeout on slow pages**
Increase with `browser_wait_for` before the action, or add `--timeout 60000` to args.

**Screenshots look wrong on Windows**
Font rendering differs from macOS — expected. Use for structure/content checks,
not pixel-perfect comparison.

**MCP server disconnects**
Playwright spins up a browser process. If Claude Code restarts mid-session,
the browser context is lost — just navigate again.
