---
name: security-audit
description: >
  Automated security scanning for Claude Code projects. Runs npm audit,
  Snyk vulnerability checks, dependency CVE scanning, and a quick OWASP
  top-10 code review pass. Wired into the closing protocol so every session
  that adds a dependency also checks if it introduced a vulnerability.
  Trigger phrases: "security audit", "npm audit", "vulnerability scan",
  "CVE", "snyk", "dependency check", "security check", "owasp".
---

# Security Audit — Automated Scanning for Claude Code

Solo founders ship fast and skip security reviews. This skill makes the
scan automatic — it runs as part of the session close, not as a separate
thing you remember to do later (and don't).

**The rule:** every session that installs a package or touches auth/API
routes ends with a security pass before the summary commit.

---

## Tools in this skill

| Tool | What it scans | Requires |
|------|--------------|----------|
| `npm audit` | Known CVEs in npm dependencies | Built-in Node.js |
| `snyk` | CVEs + license issues + code analysis | `npm install -g snyk` + auth |
| Manual OWASP pass | Code-level: injection, XSS, auth flaws | Claude reviewing your code |

---

## Install

```bash
# Snyk CLI
npm install -g snyk

# Authenticate (free tier covers open source + basic code scanning)
snyk auth
```

---

## The scan workflow

### Level 1 — Every session (30 seconds)

```bash
npm audit
```

Fix automatically where safe:
```bash
npm audit fix
```

Fix including breaking changes (review first):
```bash
npm audit fix --force
```

### Level 2 — Weekly or after adding packages (2 minutes)

```bash
# Snyk dependency scan
snyk test

# Snyk code scan (SAST — finds injection, XSS, etc.)
snyk code test

# Monitor for future vulnerabilities (registers snapshot)
snyk monitor
```

### Level 3 — Pre-launch or major feature (5 minutes)

Claude does an OWASP Top-10 pass on changed files:

```
Run a security review on these files: [list changed files].
Check specifically for:
1. SQL injection / NoSQL injection
2. XSS (unsanitized user input in DOM)
3. Broken auth (JWT not verified, session not invalidated)
4. Sensitive data exposure (secrets in code, PII in logs)
5. Insecure direct object references (user can access other users' data)
6. Missing RLS on Supabase tables
7. API routes with no auth check
```

---

## Wiring into the closing protocol

Add to your session close (before option 2 — Summarize/Save):

```
Security pass:
□ npm audit → 0 high/critical vulnerabilities
□ snyk test → clean (or known exceptions documented)
□ Any new API routes? → confirm auth check present
□ Any new Supabase tables? → confirm RLS + 4 policies
□ THEN summarize and commit
```

Add to your `/start` command:

```
If last session added packages: run npm audit before any new work.
```

---

## Supabase-specific checks

The most common security gap in Supabase projects:

```bash
# Claude: check every table in supabase/migrations/ for these patterns
# 1. RLS enabled?
grep -r "enable row level security" supabase/migrations/

# 2. Policies present for each table?
grep -r "create policy" supabase/migrations/
```

Every table should appear in both greps. A table with RLS but no policy
blocks all access. A table without RLS exposes all rows to all authenticated users.

---

## Snyk ignore (for known false positives)

```bash
# Ignore a specific vulnerability with an expiry
snyk ignore --id=SNYK-JS-EXAMPLE-12345 --expiry=2026-12-31 --reason="Dev dep only, not in prod bundle"
```

Document every ignore. Unknown ignores are future liability.

---

## Environment variable security

```bash
# Find any hardcoded secrets in code (common mistake)
grep -r "sk-ant\|sk-proj\|SUPABASE_SERVICE\|password\s*=\s*['\"]" src/ --include="*.ts" --include="*.tsx"

# Confirm .env* is in .gitignore
grep -E "^\.env" .gitignore
```

Add to Claude's pre-commit check:
```
Before committing: grep for any API keys, tokens, or passwords in changed files.
Block the commit if any are found.
```

---

## npm audit output guide

```
found N vulnerabilities (X moderate, Y high, Z critical)
```

| Severity | Action |
|----------|--------|
| Critical | Fix immediately. Block deploy. |
| High | Fix before next deploy. |
| Moderate | Fix this sprint. Document if deferring. |
| Low | Track in KNOWN_ISSUES.md. Fix in batch. |

---

## Snyk free tier limits

- Dependency scanning: unlimited
- Code scanning (SAST): 100 tests/month
- Monitoring: 200 projects

For solo SaaS: free tier is sufficient. Upgrade if you need container scanning or team features.

---

## Quick reference

```bash
npm audit                          # CVE scan, dependency tree
npm audit fix                      # Auto-fix safe upgrades
npm audit fix --force              # Fix including major version bumps
npm audit --json                   # Machine-readable output

snyk test                          # Dependency CVE scan
snyk code test                     # SAST code scan
snyk monitor                       # Register snapshot for ongoing monitoring
snyk auth                          # Authenticate with Snyk
snyk ignore --id=<id>              # Ignore a known issue
```
