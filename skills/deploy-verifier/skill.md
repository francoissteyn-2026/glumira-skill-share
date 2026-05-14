---
name: deploy-verifier
description: >
  Post-deploy verification discipline for Vercel projects. Every deploy gets
  health-checked before the session closes. Claude pulls the preview URL,
  checks HTTP status, validates critical routes, inspects build logs for
  errors, and flags regressions. Wired into the closing protocol so "ship"
  means "verified ship" not "pushed and hoped".
  Trigger phrases: "verify deploy", "check deploy", "post-deploy", "preview URL",
  "deploy check", "health check", "did it work", "vercel deploy".
---

# Deploy Verifier — Every Ship Gets Checked

Pushing to Vercel and assuming it worked is how silent regressions reach users.
This skill wires post-deploy verification into the session closing protocol.
Claude checks the deploy before you summarize and commit. Not after.

**The rule:** no session ends with a "done" if a deploy happened in that session
without a verification pass first.

---

## What gets checked

Every deploy verification covers:

```
1. Build status       — did the build succeed or fail silently?
2. HTTP status        — is the preview URL returning 200?
3. Critical routes    — do key pages load (home, auth, main feature)?
4. Console errors     — any JS errors on load?
5. Env var canary     — does the app behave like env vars are wired correctly?
```

---

## Install (Vercel CLI)

```bash
npm install -g vercel
vercel login
```

Link your project:
```bash
cd your-project
vercel link
```

---

## The verification workflow

### Step 1 — Get the latest preview URL

```bash
vercel ls --json | head -20
```

Or after a push:
```bash
vercel inspect --json $(vercel ls --json | python -c "import sys,json; d=json.load(sys.stdin); print(d[0]['url'])") 2>/dev/null
```

Or directly after deploy:
```bash
vercel deploy --prebuilt 2>&1 | tail -5
```

### Step 2 — Check build logs

```bash
# Get deployment ID from ls, then:
vercel logs <deployment-url> 2>&1 | grep -i "error\|warning\|failed" | head -20
```

### Step 3 — HTTP health check

```bash
# Check root
curl -sI https://<preview-url> | head -5

# Check critical routes
for route in / /login /dashboard /api/health; do
  status=$(curl -sI https://<preview-url>$route | head -1 | awk '{print $2}')
  echo "$route → $status"
done
```

### Step 4 — Browser spot check (with Playwright)

If Playwright is wired:
```
Navigate to <preview URL>.
Screenshot the landing page.
Check console for errors.
Navigate to /login — confirm it renders.
```

---

## Wiring into the closing protocol

Add to your session close sequence (before option 2 — Summarize/Save):

```
Pre-commit checklist:
□ vercel ls → confirm latest deploy succeeded
□ curl -I <preview-url> → 200 OK
□ Critical routes pass (/, /login, /dashboard)
□ No new console errors
□ THEN summarize and commit
```

Add to your `/start` command:

```
If last session deployed: run deploy-verifier before any new work.
Don't build on top of a broken deploy.
```

---

## Rollback when it fails

```bash
# List recent deployments
vercel ls

# Promote a previous deployment to production
vercel alias set <old-deployment-url> <your-domain>
```

Or via Vercel dashboard: Deployments → find the last good one → Promote to Production.

---

## Environment variable canary

The most common silent failure: env vars missing in preview but present locally.
Test by checking behaviour that requires a real env var:

```bash
# If you have an /api/health endpoint that returns env var status:
curl https://<preview-url>/api/health

# Or check Supabase connectivity — it'll 401 if SUPABASE_URL is missing
curl https://<preview-url>/api/ping
```

Add to your project:

```typescript
// src/app/api/health/route.ts
export async function GET() {
  return Response.json({
    supabase: !!process.env.NEXT_PUBLIC_SUPABASE_URL,
    anon_key: !!process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
    env: process.env.VERCEL_ENV ?? 'unknown'
  })
}
```

Then `curl /api/health` is the single fastest way to confirm env wiring.

---

## Vercel-specific failure modes

| Failure | Signal | Fix |
|---------|--------|-----|
| Build fails silently | `vercel ls` shows "Error" state | Check build logs: `vercel logs <url>` |
| Preview works, prod broken | Env vars differ per environment | Check Vercel dashboard → Settings → Environment Variables |
| 404 on all routes | Framework preset wrong or output dir misconfigured | Check `vercel.json` → `outputDirectory` |
| 500 on API routes | Server-side error, no client error | Check `vercel logs` for the function output |
| Functions timeout | Serverless function > 10s | Add `export const maxDuration = 30` to route |

---

## Quick reference

```bash
vercel ls                          # List recent deployments
vercel logs <url>                  # Build + function logs
vercel inspect <url>               # Full deployment metadata
vercel alias set <from> <to>       # Promote or rollback
vercel env ls                      # List env vars by environment
vercel env pull .env.local         # Pull remote env to local
```
