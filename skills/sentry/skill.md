---
name: sentry
description: >
  Production error monitoring via Sentry. Every unhandled exception, API
  failure, and performance regression gets captured, grouped, and surfaced.
  Covers SDK setup for React + Vite, source maps upload on deploy, Sentry
  MCP server for querying issues from Claude Code, and alert routing.
  Trigger phrases: "sentry", "error monitoring", "production errors",
  "error tracking", "unhandled exception", "source maps", "alerting".
---

# Sentry — Production Errors to Claude Code

Without error monitoring, you learn about production failures when a user
complains. With Sentry, you know before they do. This skill wires Sentry
into a Vite/React project and connects it to Claude Code via the Sentry
MCP server so you can query production issues without leaving the session.

---

## Install

```bash
npm install @sentry/react @sentry/vite-plugin
```

---

## Create a Sentry project

1. Sign up at [sentry.io](https://sentry.io) (free plan: 5k errors/month)
2. New Project → React
3. Copy your DSN: `https://xxxx@oyyy.ingest.sentry.io/zzzz`
4. Copy your Auth Token from Settings → Auth Tokens

---

## SDK setup

### `src/main.tsx`

```typescript
import * as Sentry from '@sentry/react'

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  environment: import.meta.env.VITE_ENV ?? 'development',
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration({
      maskAllText: true,       // HIPAA/privacy: mask user input
      blockAllMedia: false,
    }),
  ],
  tracesSampleRate: import.meta.env.PROD ? 0.1 : 1.0,
  replaysSessionSampleRate: 0.05,
  replaysOnErrorSampleRate: 1.0,  // Always capture on error
  beforeSend(event) {
    // Don't send events in development
    if (import.meta.env.DEV) return null
    return event
  },
})
```

### `vite.config.ts` — source maps upload

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import { sentryVitePlugin } from '@sentry/vite-plugin'

export default defineConfig({
  plugins: [
    react(),
    sentryVitePlugin({
      org: 'your-sentry-org',
      project: 'your-project-name',
      authToken: process.env.SENTRY_AUTH_TOKEN,
      sourcemaps: {
        assets: './dist/**',
      },
    }),
  ],
  build: {
    sourcemap: true,  // Required for readable stack traces
  },
})
```

### Environment variables

```bash
# .env
VITE_SENTRY_DSN=https://xxxx@oyyy.ingest.sentry.io/zzzz
VITE_ENV=production

# .env.local (not committed)
SENTRY_AUTH_TOKEN=sntrys_...
```

Add to Vercel: Settings → Environment Variables → add both.

---

## Error boundary (React)

Wrap your app to catch render errors with a user-friendly fallback:

```typescript
// src/components/ErrorBoundary.tsx
import * as Sentry from '@sentry/react'

export const ErrorBoundary = Sentry.withErrorBoundary(
  ({ children }) => <>{children}</>,
  {
    fallback: ({ error, resetError }) => (
      <div className="error-boundary">
        <h2>Something went wrong</h2>
        <p>The error has been reported. Try refreshing.</p>
        <button onClick={resetError}>Try again</button>
      </div>
    ),
    onError: (error) => {
      console.error('Boundary caught:', error)
    },
  }
)
```

```typescript
// src/main.tsx
root.render(
  <ErrorBoundary>
    <App />
  </ErrorBoundary>
)
```

---

## Manual error capture

```typescript
// Capture a specific error with context
Sentry.captureException(error, {
  tags: { feature: 'dose-calculator', user_tier: 'free' },
  extra: { insulinType, units, timeOfDay },
})

// Capture a message (non-error event)
Sentry.captureMessage('Payment webhook received', 'info')

// Add breadcrumbs for context
Sentry.addBreadcrumb({
  category: 'user-action',
  message: 'User submitted dose form',
  level: 'info',
  data: { units, insulinType },
})
```

---

## Sentry MCP server

Query Sentry issues directly from Claude Code:

```bash
npm install -g @sentry/mcp-server
```

Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "sentry": {
      "command": "npx",
      "args": ["-y", "@sentry/mcp-server"],
      "env": {
        "SENTRY_AUTH_TOKEN": "${SENTRY_AUTH_TOKEN}",
        "SENTRY_ORG": "your-sentry-org"
      }
    }
  }
}
```

Once wired, you can ask Claude:
- "What are the top 5 unresolved errors in Sentry from the last 7 days?"
- "Show me the stack trace for the TypeError in the dose calculator"
- "How many users were affected by the auth error yesterday?"
- "Resolve the ISSUE-123 in Sentry and add a note: fixed in deploy abc123"

---

## Alert configuration

In Sentry → Alerts → Create Alert Rule:

**Critical path alert:**
```
Condition: Error rate > 1% in 5 minutes
Filter: transaction matches /api/doses/*
Action: Send email + Slack notification
```

**New issue alert:**
```
Condition: New issue first seen
Filter: environment = production
Action: Send email immediately
```

**Performance alert:**
```
Condition: p95 response time > 3s
Filter: transaction matches /dashboard
Action: Send email
```

---

## Privacy / HIPAA considerations

For clinical or health apps:

```typescript
Sentry.init({
  // ...
  beforeSend(event) {
    // Strip PII from error events
    if (event.user) {
      delete event.user.email
      delete event.user.username
      event.user.id = hashUserId(event.user.id)
    }
    return event
  },
  integrations: [
    Sentry.replayIntegration({
      maskAllText: true,        // Mask all visible text
      maskAllInputs: true,      // Mask all form inputs
      blockAllMedia: true,      // Block all images/video
    }),
  ],
})
```

Never send raw dose values, BG readings, or any clinical data to Sentry.
Use masked replays and anonymized user IDs.

---

## Wiring into the closing protocol

After first setup — add to CLAUDE.md:

```markdown
## Error monitoring
Sentry is active on production. After every deploy:
1. Check Sentry for new issues introduced by this deploy
2. Any critical/high errors = fix before next feature
3. Resolution: link commit hash in Sentry issue comment
```

---

## Quick reference

```bash
# Verify Sentry is receiving events (use Sentry Test button in dashboard)
# Or trigger manually in browser console:
# Sentry.captureMessage('test')

# Check source map upload worked:
# Open a Sentry error → stack trace should show your source code, not minified JS
```
