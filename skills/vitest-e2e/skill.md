---
name: vitest-e2e
description: >
  Test-driven discipline for Vite/React projects using Vitest (unit/integration)
  and Playwright (E2E). Claude generates tests alongside code, runs the suite
  before every commit, and interprets failures. Covers the full stack: unit
  functions, React component tests, API route tests, and E2E critical paths.
  Wired into the closing protocol so no commit ships with a red test suite.
  Trigger phrases: "vitest", "write tests", "test this", "e2e test", "unit test",
  "test coverage", "failing test", "tdd", "test suite".
---

# Vitest + E2E — Test Discipline for Claude Code

Tests that Claude doesn't run are tests that don't exist. This skill makes
the test suite part of the session workflow — Claude writes tests alongside
code, runs them before committing, and stops at red.

**The rule:** no commit ships with a failing test. No new feature ships
without at least one test covering the critical path.

---

## Install

### Vitest (unit + component tests)

```bash
npm install -D vitest @vitest/ui happy-dom @testing-library/react @testing-library/user-event
```

### Playwright (E2E)

```bash
npm install -D @playwright/test
npx playwright install chromium
```

---

## Config

### `vitest.config.ts`

```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'happy-dom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      reporter: ['text', 'html'],
      exclude: ['node_modules/', 'src/test/']
    }
  }
})
```

### `src/test/setup.ts`

```typescript
import '@testing-library/jest-dom'
```

### `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  use: {
    baseURL: 'http://localhost:5173',
    screenshot: 'only-on-failure',
    trace: 'on-first-retry',
  },
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: true,
  },
})
```

### `package.json` scripts

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "e2e": "playwright test",
    "e2e:headed": "playwright test --headed",
    "e2e:ui": "playwright test --ui"
  }
}
```

---

## The TDD workflow with Claude

### Pattern: write test first, then implementation

```
1. Tell Claude the behaviour you want.
2. Claude writes a failing test.
3. Claude writes the implementation that makes it pass.
4. Run: npm test
5. Green → commit. Red → fix before anything else.
```

### Example prompt

```
Write a Vitest test for the calculateIOB function.
Test cases:
- Returns 0 when no doses in the last 4 hours
- Returns the correct IOB for a single dose at peak
- Sums multiple overlapping doses correctly
Then write the implementation that passes all three.
```

---

## Test types and when to use each

### Unit test — pure functions

```typescript
// src/lib/iob.test.ts
import { describe, it, expect } from 'vitest'
import { calculateIOB } from './iob'

describe('calculateIOB', () => {
  it('returns 0 with no recent doses', () => {
    expect(calculateIOB([], new Date())).toBe(0)
  })

  it('returns full dose at injection time', () => {
    const now = new Date()
    const doses = [{ units: 4, injectedAt: now, insulinType: 'Humalog' }]
    expect(calculateIOB(doses, now)).toBeCloseTo(4, 1)
  })
})
```

### Component test — React + user interaction

```typescript
// src/components/DoseForm.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { DoseForm } from './DoseForm'

describe('DoseForm', () => {
  it('submits dose when form is filled', async () => {
    const onSubmit = vi.fn()
    render(<DoseForm onSubmit={onSubmit} />)

    await userEvent.type(screen.getByLabelText('Units'), '4')
    await userEvent.click(screen.getByRole('button', { name: 'Log Dose' }))

    expect(onSubmit).toHaveBeenCalledWith({ units: 4 })
  })
})
```

### API route test — edge functions / server routes

```typescript
// src/app/api/doses/route.test.ts
import { POST } from './route'

describe('POST /api/doses', () => {
  it('returns 401 when not authenticated', async () => {
    const req = new Request('http://localhost/api/doses', {
      method: 'POST',
      body: JSON.stringify({ units: 4 }),
    })
    const res = await POST(req)
    expect(res.status).toBe(401)
  })
})
```

### E2E test — critical user flows

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test('user can log in and see dashboard', async ({ page }) => {
  await page.goto('/login')
  await page.fill('[name="email"]', 'test@example.com')
  await page.fill('[name="password"]', 'Test1234!')
  await page.click('button[type="submit"]')

  await expect(page).toHaveURL('/dashboard')
  await expect(page.locator('h1')).toContainText('Dashboard')
})
```

---

## Wiring into the closing protocol

Add to your session close (before option 2 — Summarize/Save):

```
Test gate:
□ npm test → all pass (0 failing)
□ New code has at least 1 test covering the critical path
□ If E2E affected: npm run e2e → critical paths pass
□ THEN summarize and commit
```

Add to CLAUDE.md:
```markdown
## Test gate (non-negotiable)
Every commit passes: npm test
New features require: at least 1 Vitest test on the main path
Auth/payment/data routes require: unit + E2E test before deploy
```

---

## Coverage targets (pragmatic solo founder)

| Layer | Target | Why |
|-------|--------|-----|
| Pure functions (calculations, formatters) | 90%+ | High value, easy to test |
| React components | 60%+ | Test behaviour, not implementation |
| API routes | 80%+ | Auth and error paths are critical |
| E2E critical paths | 100% | Login, main feature, payment |

Don't chase 100% overall. Chase 100% on the paths that break users.

---

## Quick reference

```bash
npm test                           # Run all Vitest tests once
npm run test:watch                 # Watch mode during development
npm run test:coverage              # Coverage report → coverage/index.html
npm run e2e                        # Run Playwright E2E (headless)
npm run e2e:headed                 # E2E with browser visible (debugging)
npx vitest run src/lib/iob.test.ts # Run a single file
npx playwright test e2e/auth.spec.ts # Run a single E2E spec
```
