# Performance Engineering Reference

Core Web Vitals, bundle optimization, rendering strategy, and animation performance.
Last verified: March 2026.

## Table of Contents
1. Core Web Vitals Budgets
2. Bundle Optimization
3. Asset Optimization
4. Rendering Strategy
5. Animation Performance
6. Monitoring & Measurement

---

## 1. Core Web Vitals Budgets

### Target Metrics

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP (Largest Contentful Paint) | < 2.5s | 2.5-4.0s | > 4.0s |
| INP (Interaction to Next Paint) | < 200ms | 200-500ms | > 500ms |
| CLS (Cumulative Layout Shift) | < 0.1 | 0.1-0.25 | > 0.25 |

**Target: All three metrics in "Good" range.** LCP < 2.0s is the aspirational target for
premium experiences.

### What Impacts Each Metric

**LCP:** Hero images, web fonts, large text blocks, server response time.
- Fix: Preload hero image, use `next/image` with `priority`, optimize server response,
  preload critical fonts, avoid render-blocking CSS.

**INP:** JavaScript execution blocking the main thread during interactions.
- Fix: Keep event handlers under 50ms, use `startTransition` for non-urgent updates,
  break long tasks with `scheduler.yield()`, avoid synchronous layout (forced reflow).

**CLS:** Elements shifting after initial render.
- Fix: Set explicit `width`/`height` on images/videos, use `font-display: swap` with
  `size-adjust` on fallback fonts, reserve space for dynamic content, avoid injecting
  content above existing content.

---

## 2. Bundle Optimization

### Code Splitting Strategy

**Route-based splitting (automatic in Next.js):** Each page is a separate chunk.

**Component-level splitting for heavy libraries:**
```jsx
import dynamic from "next/dynamic";

// GSAP-heavy component — only load when needed
const ScrollSection = dynamic(() => import("./ScrollSection"), {
  ssr: false,  // Disable SSR for browser-only animation code
  loading: () => <div className="h-screen" />,  // Reserve space
});

// R3F scene — always client-only, always lazy
const ThreeScene = dynamic(() => import("./ThreeScene"), { ssr: false });
```

**Tree-shaking GSAP:**
Import only the plugins you use:
```js
// Good — tree-shakeable
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

// Bad — imports everything
import { gsap, ScrollTrigger, DrawSVGPlugin, ... } from "gsap/all";
```

**Motion bundle splitting:**
```jsx
import { LazyMotion, domAnimation, m } from "motion/react";

// domAnimation = ~15kb (animate, exit, hover, tap, focus, layout)
// domMax = ~30kb (adds drag, pan, layoutId, AnimatePresence)

<LazyMotion features={domAnimation}>
  <m.div animate={{ opacity: 1 }} />
</LazyMotion>
```

### Bundle Size Budgets

| Category | Budget | Includes |
|----------|--------|----------|
| Framework | ~85kb | React + Next.js runtime |
| UI Components | ~30kb | shadcn/ui used components |
| Animations (Tier 2) | ~15kb | Motion (domAnimation) |
| Animations (Tier 3) | ~45kb | GSAP core + ScrollTrigger + Lenis |
| Animations (Tier 4) | ~200kb | + Three.js + R3F + drei |
| Styles | ~15kb | Tailwind (only used utilities) |
| **Total (Tier 3)** | **~190kb** | Gzipped JS + CSS |

These are gzipped sizes. Monitor with `next build` output or `npx next-bundle-analyzer`.

---

## 3. Asset Optimization

### Image Strategy

**Format priority:** AVIF > WebP > PNG/JPEG

**next/image (always use for Next.js projects):**
```jsx
import Image from "next/image";

<Image
  src="/hero.jpg"
  alt="Hero image description"
  width={1920}
  height={1080}
  priority              // Preload — use for above-the-fold images only
  quality={85}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  placeholder="blur"    // Show blurred preview during load
  blurDataURL={blurUrl} // Base64 tiny version
/>
```

**Rules:**
- `priority` on LCP image (hero) ONLY — never on more than 1-2 images per page
- Always set `sizes` for responsive images — prevents downloading oversized images
- Use `placeholder="blur"` for images below the fold
- Let Next.js handle format conversion (AVIF/WebP) — don't pre-convert

### Font Strategy

1. **Self-host** variable fonts (single file, all weights)
2. **Preload** critical fonts in `<head>`:
   ```html
   <link rel="preload" href="/fonts/display.woff2" as="font" type="font/woff2" crossorigin />
   ```
3. **Subset** to needed character ranges (latin: ~20kb vs full Unicode: ~200kb)
4. **size-adjust** on fallback to minimize CLS:
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
5. Use `next/font` for automatic optimization in Next.js projects

### SVG Optimization

- Run through SVGO before using: `npx svgo icon.svg`
- Inline SVGs for icons (avoids HTTP requests)
- Use `<symbol>` + `<use>` for repeated SVGs (sprite approach)
- Set `aria-hidden="true"` on decorative SVGs

---

## 4. Rendering Strategy

### Server Component vs Client Component Boundary

```
Server Components (default — no "use client"):
├── Layouts
├── Data fetching
├── Static content
├── Metadata
└── Wrapping Client Components

Client Components ("use client"):
├── Interactive elements (buttons, forms, modals)
├── Animation components (GSAP, Motion, Lenis)
├── Browser API usage (localStorage, IntersectionObserver)
├── Zustand stores
└── Event handlers
```

**Push "use client" as deep as possible.** Don't make an entire page client-side because
one button needs onClick. Extract the interactive part into a small Client Component.

### Streaming & Suspense

Wrap slow data fetches in Suspense for progressive rendering:
```jsx
// app/dashboard/page.tsx (Server Component)
import { Suspense } from "react";
import { AnalyticsSkeleton } from "./skeletons";

export default function DashboardPage() {
  return (
    <main>
      <h1>Dashboard</h1>
      {/* This renders immediately */}
      <QuickStats />

      {/* This streams in when ready */}
      <Suspense fallback={<AnalyticsSkeleton />}>
        <AnalyticsChart />
      </Suspense>
    </main>
  );
}
```

### Cache Components ("use cache")

Next.js 16's explicit caching — use for expensive operations:
```tsx
"use cache";

export default async function PricingPage() {
  const plans = await fetchPlans(); // Cached across requests
  return <PricingGrid plans={plans} />;
}
```

---

## 5. Animation Performance

### The Golden Rule

**Only animate `transform` and `opacity`.** These are compositor-only properties — they
skip layout and paint, running entirely on the GPU.

Properties that trigger layout (AVOID animating):
- `width`, `height`, `padding`, `margin`, `top`, `left`, `right`, `bottom`
- `font-size`, `border-width`, `line-height`

Properties that trigger paint (AVOID if possible):
- `background-color`, `color`, `box-shadow`, `border-color`
- These are acceptable for simple color transitions but expensive in scroll-linked animations

### will-change Strategy

```css
/* Apply ONLY right before animation starts */
.about-to-animate { will-change: transform, opacity; }

/* Remove AFTER animation completes */
.done-animating { will-change: auto; }
```

**Rules:**
- Never apply `will-change` to more than ~5 elements simultaneously
- Never use `will-change: transform` on ancestors of `position: fixed` elements (breaks fixed positioning)
- GSAP applies `will-change` automatically for active animations — don't double-apply

### Single RAF Loop

When using Lenis + GSAP, ensure only ONE requestAnimationFrame loop runs:
```js
// Lenis driven by GSAP's ticker (CORRECT — single loop)
const lenis = new Lenis(); // No autoRaf
gsap.ticker.add((time) => lenis.raf(time * 1000));

// BAD — two competing loops
const lenis = new Lenis({ autoRaf: true });
gsap.ticker.add(() => { /* separate loop */ });
```

### Debounce Resize

ScrollTrigger already debounces resize with a 200ms gap. If you must handle resize:
```js
let resizeTimeout;
window.addEventListener("resize", () => {
  clearTimeout(resizeTimeout);
  resizeTimeout = setTimeout(() => {
    ScrollTrigger.refresh();
  }, 250);
});
```

### Mobile Performance

- Disable `smoothTouch` in Lenis (`smoothTouch: false`) — native touch scrolling is smoother
- Reduce parallax intensity on mobile (10% vs 30% offset)
- Use `matchMedia` to skip expensive animations on low-power devices
- Limit ScrollTrigger instances — each one adds a scroll listener
- Use `fastScrollEnd: true` on ScrollTrigger for quicker cleanup during fast scrolls

---

## 6. Monitoring & Measurement

### Development

- **Lighthouse** — Run in Chrome DevTools (Incognito mode, no extensions)
- **Web Vitals extension** — Real-time CWV overlay in browser
- **React DevTools Profiler** — Identify unnecessary re-renders
- **Next.js build output** — Check route sizes, shared chunks

### Production

- **Vercel Analytics** — Built-in Web Vitals tracking for Vercel deployments
- **Google Search Console** — Core Web Vitals report from real users
- **web-vitals npm package** — Send metrics to your own analytics:
  ```js
  import { onLCP, onINP, onCLS } from "web-vitals";
  onLCP(console.log);
  onINP(console.log);
  onCLS(console.log);
  ```

### Performance Checklist Before Deploy

- [ ] Lighthouse Performance score > 90
- [ ] LCP < 2.5s on 3G throttled
- [ ] No layout shift visible during page load
- [ ] Fonts preloaded, no FOUT/FOIT flash
- [ ] Images use next/image with proper sizes attribute
- [ ] Heavy animation libs dynamically imported
- [ ] prefers-reduced-motion tested and working
- [ ] Bundle size within budget (check next build output)
