# Performance Engineering Reference
Core Web Vitals, bundle optimization, rendering strategy, animation performance. Last verified: March 2026.

## Contents
1. Core Web Vitals Budgets
2. Bundle Optimization
3. Asset Optimization
4. Rendering Strategy
5. Animation Performance
6. Monitoring

---

## 1. Core Web Vitals Budgets

| Metric | Good | Needs Work | Poor |
|---|---|---|---|
| LCP (Largest Contentful Paint) | < 2.5s | 2.5-4.0s | > 4.0s |
| INP (Interaction to Next Paint) | < 200ms | 200-500ms | > 500ms |
| CLS (Cumulative Layout Shift) | < 0.1 | 0.1-0.25 | > 0.25 |

Target all three "Good". Aspirational LCP < 2.0s.

**What impacts each:**

**LCP:** Hero images, web fonts, large text, server response time.
Fix: preload hero image · `next/image` with `priority` · preload critical fonts · no render-blocking CSS

**INP:** JS blocking main thread during interactions.
Fix: event handlers < 50ms · `startTransition` for non-urgent updates · `scheduler.yield()` for long tasks · avoid forced reflow

**CLS:** Elements shifting after initial render.
Fix: explicit `width`/`height` on images/videos · `font-display: swap` + `size-adjust` on fallback · reserve space for dynamic content · never inject content above existing

---

## 2. Bundle Optimization

**Dynamic imports for heavy libs:**
```jsx
import dynamic from "next/dynamic";

const ScrollSection = dynamic(() => import("./ScrollSection"), {
  ssr: false,
  loading: () => <div className="h-screen" />, // Reserve space to prevent CLS
});
const ThreeScene = dynamic(() => import("./ThreeScene"), { ssr: false });
```

**Tree-shaking GSAP:**
```js
// Good
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

// Bad — imports everything
import { gsap, ScrollTrigger, DrawSVGPlugin } from "gsap/all";
```

**Motion bundle splitting:**
```jsx
import { LazyMotion, domAnimation, m } from "motion/react";
// domAnimation ~15kb · domMax ~30kb
<LazyMotion features={domAnimation}>
  <m.div animate={{ opacity: 1 }} />
</LazyMotion>
```

**Bundle size budgets (gzipped):**

| Category | Budget |
|---|---|
| Framework (React + Next.js) | ~85kb |
| UI Components (shadcn used) | ~30kb |
| Animations Tier 2 (Motion) | ~15kb |
| Animations Tier 3 (GSAP + Lenis) | ~45kb |
| Animations Tier 4 (+ Three.js + R3F) | ~200kb |
| Styles (Tailwind used utilities) | ~15kb |
| **Total Tier 3** | **~190kb** |

Monitor with `next build` output or `npx next-bundle-analyzer`.

---

## 3. Asset Optimization

**Images — format priority: AVIF > WebP > PNG/JPEG**

```jsx
import Image from "next/image";
<Image
  src="/hero.jpg" alt="Hero" width={1920} height={1080}
  priority           // LCP image only — never more than 1-2 per page
  quality={85}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  placeholder="blur"
  blurDataURL={blurUrl}
/>
```

Rules: `priority` on LCP only · always set `sizes` · `placeholder="blur"` below fold · let Next.js handle format conversion

**Fonts:**
1. Self-host variable fonts (one file, all weights)
2. Preload critical: `<link rel="preload" href="/fonts/display.woff2" as="font" crossorigin />`
3. Subset to latin (~20kb vs full Unicode ~200kb)
4. `size-adjust` on fallback to minimize CLS:
```css
@font-face {
  font-family: "Display Fallback";
  src: local("Arial");
  size-adjust: 105%;
  ascent-override: 90%;
  descent-override: 22%;
  line-gap-override: 0%;
}
```
5. Use `next/font` in Next.js projects (automatic optimization)

**SVG:** Run through SVGO · inline for icons (no HTTP requests) · `<symbol>` + `<use>` for repeated · `aria-hidden="true"` on decorative

---

## 4. Rendering Strategy

```
Server Components (default):
├── Layouts, data fetching, static content, metadata
└── Wrapping Client Components

Client Components ("use client"):
├── Interactive elements (buttons, forms, modals)
├── Animation components (GSAP, Motion, Lenis)
├── Browser APIs (localStorage, IntersectionObserver)
├── Zustand stores, event handlers
```

Push `"use client"` as deep as possible.

**Streaming + Suspense:**
```jsx
// app/dashboard/page.tsx (Server Component)
import { Suspense } from "react";
export default function DashboardPage() {
  return (
    <main>
      <QuickStats />  {/* Renders immediately */}
      <Suspense fallback={<AnalyticsSkeleton />}>
        <AnalyticsChart />  {/* Streams when ready */}
      </Suspense>
    </main>
  );
}
```

**`"use cache"` (Next.js 16 — explicit opt-in):**
```tsx
"use cache";
export default async function PricingPage() {
  const plans = await fetchPlans(); // Cached across requests
  return <PricingGrid plans={plans} />;
}
```

---

## 5. Animation Performance

**Golden rule: animate `transform` and `opacity` only.** GPU-only, skips layout and paint.

Avoid animating: `width` `height` `padding` `margin` `top` `left` (layout) · `background-color` `box-shadow` `color` (paint, expensive in scroll-linked)

**`will-change`:**
```css
.about-to-animate { will-change: transform, opacity; }
.done-animating   { will-change: auto; }
```
Rules: max ~5 elements simultaneously · never on ancestors of `position: fixed` · GSAP applies automatically — don't double-apply

**Single RAF loop (Lenis + GSAP):**
```js
// Correct — one loop
const lenis = new Lenis(); // no autoRaf
gsap.ticker.add((time) => lenis.raf(time * 1000));

// Wrong — two competing loops
const lenis = new Lenis({ autoRaf: true });
gsap.ticker.add(() => { /* separate loop */ });
```

**Mobile:** `smoothTouch: false` in Lenis · reduce parallax intensity (10% not 30%) · `matchMedia` to skip expensive animations · limit ScrollTrigger instances · `fastScrollEnd: true` on ScrollTrigger

---

## 6. Monitoring

**Dev:** Lighthouse (Chrome DevTools, Incognito) · Web Vitals extension (real-time overlay) · React DevTools Profiler (re-renders) · `next build` output (route sizes)

**Production:**
```js
import { onLCP, onINP, onCLS } from "web-vitals";
onLCP(console.log); onINP(console.log); onCLS(console.log);
```
Vercel Analytics (built-in CWV for Vercel) · Google Search Console (real user CWV)

**Pre-deploy checklist:**
- [ ] Lighthouse Performance > 90
- [ ] LCP < 2.5s on 3G throttled
- [ ] No visible layout shift during load
- [ ] Fonts preloaded, no FOUT flash
- [ ] Images use `next/image` with `sizes`
- [ ] Heavy animation libs dynamically imported
- [ ] `prefers-reduced-motion` tested
- [ ] Bundle within budget (check `next build` output)
