---
name: animejs
description: >
  Use when adding, editing, or debugging animations in GluMira V7 using Anime.js 4.x.
  Covers the v4 ESM API (animate, timeline, scroll, draggable, spring physics),
  React integration patterns, and GluMira-specific animation conventions.
---

# Anime.js 4.x — GluMira V7 Skill

## Package

```
animejs@4.4.1  (installed in C:\glumira-v7)
```

## Import — always use named ESM imports

```ts
// Motion primitives
import { animate, createTimeline, stagger } from 'animejs'

// Scroll and drag (lazy-import if not used at page load)
import { onScroll }   from 'animejs'
import { createDraggable } from 'animejs'

// Easing utilities
import { spring, eases } from 'animejs'
```

Never import the default export (`import anime from 'animejs'`) — that is the v3 API and will throw in v4.

---

## Core API — v4 vs v3 cheat-sheet

| v3 | v4 |
|----|----|
| `anime({ targets, ... })` | `animate(targets, params)` |
| `anime.timeline()` | `createTimeline(params)` |
| `anime.stagger(val)` | `stagger(val, options?)` |
| `anime.remove(targets)` | `animation.cancel()` or `animation.revert()` |
| `anime.running` | not needed — use returned instance |
| `anime.easings` | `eases` named export |
| `complete` callback | `onComplete` param |
| `update` callback | `onUpdate` param |

---

## animate() — essential params

```ts
const anim = animate(
  '.card',           // target: CSS selector | Element | NodeList | array
  {
    translateY: [-20, 0],   // from → to
    opacity:    [0, 1],
    duration:   400,         // ms
    delay:      stagger(60), // 60ms between each element
    easing:     'easeOutExpo',
    loop:       false,
    autoplay:   true,
    onComplete: (a) => { /* a = animation instance */ }
  }
)

// Control
anim.pause()
anim.play()
anim.restart()
anim.cancel()   // stops + cleans up
anim.revert()   // stops + resets to original state
```

---

## React integration pattern — useAnimate hook

```tsx
import { useEffect, useRef } from 'react'
import { animate }           from 'animejs'

export function useAnimate(
  selector: string,
  params: Parameters<typeof animate>[1],
  deps: React.DependencyList = []
) {
  const animRef = useRef<ReturnType<typeof animate> | null>(null)

  useEffect(() => {
    animRef.current = animate(selector, { ...params, autoplay: true })
    return () => { animRef.current?.cancel() }
  // eslint-disable-next-line react-hooks/exhaustive-deps
  }, deps)

  return animRef
}
```

For ref-targeted animations (avoids selector leakage across components):

```tsx
import { useEffect, useRef } from 'react'
import { animate }           from 'animejs'

function FadeInCard({ children }: { children: React.ReactNode }) {
  const ref = useRef<HTMLDivElement>(null)

  useEffect(() => {
    if (!ref.current) return
    const anim = animate(ref.current, {
      opacity:    [0, 1],
      translateY: [-12, 0],
      duration:   350,
      easing:     'easeOutCubic',
    })
    return () => anim.cancel()
  }, [])

  return <div ref={ref}>{children}</div>
}
```

**Rule:** Always return the cleanup function (`anim.cancel()`) from useEffect. Failing to do so causes animations to run on unmounted DOM nodes and leaks memory.

---

## Timeline — sequencing multiple animations

```ts
import { createTimeline, stagger } from 'animejs'

const tl = createTimeline({ defaults: { duration: 300, easing: 'easeOutQuart' } })

tl
  .add('.hero-title',    { opacity: [0, 1], translateY: [-24, 0] })
  .add('.hero-subtitle', { opacity: [0, 1], translateY: [-16, 0] }, '-=150') // overlap 150ms
  .add('.hero-cta',      { opacity: [0, 1], scale: [0.9, 1] }, '+=50')
  .add('.hero-image',    { opacity: [0, 1], translateX: [40, 0] }, 0) // starts at t=0

// Control the whole sequence
tl.pause()
tl.play()
tl.seek(500)  // jump to 500ms
```

---

## Spring physics

```ts
import { animate, spring } from 'animejs'

animate('.modal', {
  scale:   [0.8, 1],
  opacity: [0,   1],
  easing:  spring({ stiffness: 180, damping: 12, mass: 1 }),
  duration: 600,
})
```

---

## Scroll-triggered animation

```ts
import { animate, onScroll } from 'animejs'

animate('.stat-number', {
  innerHTML: [0, 2847],   // count-up
  duration:  1200,
  easing:    'easeOutExpo',
  modifier:  (v) => Math.round(v).toLocaleString(),
  autoplay:  onScroll({ enter: 'bottom 80%' }),  // fires when el enters viewport
})
```

---

## GluMira conventions

### Where animations live
- Page-entry animations: co-located in the page component via `useEffect`
- Reusable micro-interactions: `src/components/ui/animated/` (create this dir)
- Chart entrance animations: only on marketing pages — NEVER on clinical data charts (IOBHunterChart, BasalActivityChart are Insulin-Lock protected)

### What to animate
- Page section reveals (opacity + translateY, 300–450ms)
- Card hovers (scale 1→1.02, 150ms, easeOutCubic)
- Modal/sheet entry (scale 0.95→1 + opacity, spring)
- Count-up numbers on marketing stats
- Mira owl entrance on landing

### What NOT to animate
- IOB curves or basal charts — any animation on clinical data requires APPROVE-INSULIN-EDIT and Insulin Lock read
- Loading spinners — use shadcn Skeleton
- Navigation transitions — page router handles these

### Performance rules
- Animate only `transform` and `opacity` where possible (compositor-only, no layout thrash)
- Use `will-change: transform` sparingly — only on elements that definitely animate
- Lazy-import `onScroll` and `createDraggable` — they add ~8KB each
- No animation on `prefers-reduced-motion: reduce` — wrap with:

```ts
const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches
if (!prefersReduced) {
  animate(el, { ... })
}
```

---

## Debugging

```ts
// Log all active animations
import { engine } from 'animejs'
console.log(engine.activeInstances)

// Check if element is being animated
import { getAnimatables } from 'animejs'
console.log(getAnimatables('.my-el'))
```

---

## TypeScript types

Anime.js 4 ships its own `.d.ts` — no `@types/animejs` needed (that's v3).
If TS complains about a specific param, check the `AnimationParams` interface:

```ts
import type { AnimationParams, TimelineParams } from 'animejs'
```
