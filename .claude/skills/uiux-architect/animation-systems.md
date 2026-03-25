# Animation Systems Reference

This is the most critical reference file. Read it fully before implementing any animation above
Tier 1. Last verified: March 2026.

## Table of Contents
1. GSAP 3.14+ (ScrollTrigger, ScrollSmoother, useGSAP)
2. Lenis 1.3.x (Smooth Scroll)
3. Lenis + GSAP Integration (Recommended Default)
4. Motion 12.x (formerly Framer Motion)
5. React Three Fiber (R3F) v9/v10
6. CSS-Native Scroll Animations
7. Combination Recipes
8. Animation Performance Rules

---

## 1. GSAP 3.14+

**Status (2026):** Completely free for all uses including commercial, after Webflow's acquisition
of GreenSock. All plugins (ScrollTrigger, ScrollSmoother, DrawSVG, MorphSVG, SplitText, etc.)
are included at no cost under the standard license.

**Installation:**
```bash
npm install gsap @gsap/react
```

**React Integration — useGSAP hook:**
The `@gsap/react` package provides `useGSAP()`, a drop-in replacement for `useEffect` /
`useLayoutEffect` that automatically handles GSAP cleanup (kills tweens, ScrollTriggers, etc.)
when the component unmounts.

```jsx
import { useGSAP } from "@gsap/react";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

function HeroSection() {
  const containerRef = useRef(null);

  useGSAP(() => {
    // Everything in here auto-cleans on unmount
    gsap.from(".hero-title", {
      y: 100,
      opacity: 0,
      duration: 1,
      ease: "power3.out",
    });

    gsap.to(".hero-image", {
      yPercent: -20,
      scrollTrigger: {
        trigger: ".hero-section",
        start: "top top",
        end: "bottom top",
        scrub: true,
      },
    });
  }, { scope: containerRef }); // Scope limits GSAP selectors to this container

  return <div ref={containerRef}>...</div>;
}
```

**ScrollTrigger Core Patterns:**

Pin a section while animation plays:
```js
gsap.timeline({
  scrollTrigger: {
    trigger: ".section",
    start: "top top",
    end: "+=200%",    // Section stays pinned for 2x viewport heights of scroll
    pin: true,
    scrub: 1,         // 1-second smooth catch-up (true = instant sync)
  },
})
.to(".title", { opacity: 1, y: 0 })
.to(".image", { scale: 1.2 })
.to(".text", { opacity: 1 });
```

Snap to sections:
```js
ScrollTrigger.create({
  trigger: ".sections-container",
  start: "top top",
  end: "bottom bottom",
  snap: 1 / (numSections - 1),  // Snap evenly across sections
});
```

Horizontal scroll via vertical scrolling:
```js
gsap.to(".horizontal-wrapper", {
  xPercent: -100 * (panels.length - 1) / panels.length,
  ease: "none",
  scrollTrigger: {
    trigger: ".horizontal-container",
    pin: true,
    scrub: 1,
    end: () => "+=" + document.querySelector(".horizontal-wrapper").scrollWidth,
  },
});
```

Staggered reveal on scroll:
```js
gsap.from(".card", {
  y: 60,
  opacity: 0,
  stagger: 0.15,
  duration: 0.8,
  ease: "power2.out",
  scrollTrigger: {
    trigger: ".cards-grid",
    start: "top 80%",
  },
});
```

**Animating CSS variables (advanced technique):**
Instead of animating properties directly, animate a CSS `--progress` variable and use it in CSS
for multi-property synchronization:
```js
gsap.to(".element", {
  "--progress": 1,
  scrollTrigger: { trigger: ".element", scrub: true },
});
```
```css
.element {
  transform: scale(calc(1 + var(--progress) * 0.5));
  opacity: calc(0.3 + var(--progress) * 0.7);
}
```

**prefers-reduced-motion handling:**
```js
const prefersReduced = window.matchMedia("(prefers-reduced-motion: reduce)");
if (prefersReduced.matches) {
  // Show final state immediately, no animation
  gsap.set(".hero-title", { opacity: 1, y: 0 });
} else {
  gsap.from(".hero-title", { opacity: 0, y: 100, duration: 1 });
}
```

---

## 2. Lenis 1.3.x

**Status (2026):** Package renamed from `@studio-freight/lenis` to just `lenis`. The old package
is deprecated. Latest stable: 1.3.20.

**Installation:**
```bash
npm install lenis
```

**Basic setup (standalone, no GSAP):**
```js
import Lenis from "lenis";
import "lenis/dist/lenis.css";   // Required CSS

const lenis = new Lenis({
  autoRaf: true,   // Handles requestAnimationFrame automatically
});
```

**Required CSS (include in global styles):**
```css
html.lenis { height: auto; }
.lenis.lenis-smooth { scroll-behavior: auto !important; }
.lenis.lenis-smooth [data-lenis-prevent] { overscroll-behavior: contain; }
.lenis.lenis-stopped { overflow: hidden; }
.lenis.lenis-scrolling iframe { pointer-events: none; }
```

**Nested scrollable elements:**
Use `data-lenis-prevent` attribute on any element that should use native scrolling:
```html
<div data-lenis-prevent>This scrolls natively inside Lenis</div>
<div data-lenis-prevent-wheel>Prevents wheel only</div>
<div data-lenis-prevent-touch>Prevents touch only</div>
```

Or use `allowNestedScroll: true` option (less performant but automatic).

**Next.js App Router integration:**
```jsx
// app/lenis-provider.jsx
"use client";
import { ReactLenis } from "lenis/react";

export default function LenisProvider({ children }) {
  return (
    <ReactLenis root options={{ autoRaf: true }}>
      {children}
    </ReactLenis>
  );
}

// app/layout.jsx
import LenisProvider from "./lenis-provider";
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <LenisProvider>{children}</LenisProvider>
      </body>
    </html>
  );
}
```

**Key options:**
- `lerp: 0.1` — Smoothness (0.05 = very smooth, 0.5 = snappy). Default: 0.1
- `wheelMultiplier: 1` — Scroll speed multiplier for mouse wheel
- `smoothTouch: false` — Keep `false` for better mobile UX (native touch is usually better)
- `infinite: false` — Infinite looping scroll
- `orientation: "vertical"` — `"vertical"`, `"horizontal"`, or `"both"`

---

## 3. Lenis + GSAP Integration (Recommended Default for Tier 3)

This is the industry-standard combo. Lenis handles the smooth scroll physics. GSAP ScrollTrigger
handles the animation triggering. They sync through GSAP's ticker.

**Why Lenis + ScrollTrigger instead of ScrollSmoother?**
ScrollSmoother requires a specific DOM wrapper structure (`#smooth-wrapper` > `#smooth-content`)
and reparents your content. Lenis works with your existing DOM — no restructuring needed. Lenis
also plays better with position: absolute/fixed elements and non-standard layouts.

**Setup:**
```js
import Lenis from "lenis";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

// Initialize Lenis WITHOUT autoRaf (GSAP ticker drives it)
const lenis = new Lenis();

// Sync Lenis scroll position with ScrollTrigger
lenis.on("scroll", ScrollTrigger.update);

// Drive Lenis from GSAP's ticker (single RAF loop, better perf)
gsap.ticker.add((time) => {
  lenis.raf(time * 1000);
});

// Prevent GSAP from smoothing lag (Lenis already handles this)
gsap.ticker.lagSmoothing(0);
```

**React/Next.js setup with both:**
```jsx
"use client";
import { useEffect, useRef } from "react";
import Lenis from "lenis";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { useGSAP } from "@gsap/react";

gsap.registerPlugin(ScrollTrigger);

export default function SmoothScrollProvider({ children }) {
  const lenisRef = useRef(null);

  useEffect(() => {
    const lenis = new Lenis();
    lenisRef.current = lenis;

    lenis.on("scroll", ScrollTrigger.update);
    gsap.ticker.add((time) => lenis.raf(time * 1000));
    gsap.ticker.lagSmoothing(0);

    return () => {
      lenis.destroy();
      gsap.ticker.remove(lenis.raf);
    };
  }, []);

  return <>{children}</>;
}
```

---

## 4. Motion 12.x (formerly Framer Motion)

**Status (2026):** Rebranded from "Framer Motion" to "Motion". Import path changed. Now supports
React, Vue, and vanilla JS. Uses Web Animations API + ScrollTimeline natively for 120fps, falls
back to JS for spring physics and gesture tracking.

**Installation:**
```bash
npm install motion
```

**Import (CRITICAL — do NOT import from framer-motion):**
```jsx
import { motion, AnimatePresence, useScroll, useTransform } from "motion/react";
```

**Core patterns:**

Layout animations:
```jsx
<motion.div layout layoutId="card">
  {/* Automatically animates position/size changes */}
</motion.div>
```

Exit animations:
```jsx
<AnimatePresence mode="wait">
  {show && (
    <motion.div
      key="modal"
      initial={{ opacity: 0, scale: 0.95 }}
      animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.95 }}
      transition={{ type: "spring", damping: 25, stiffness: 300 }}
    />
  )}
</AnimatePresence>
```

Scroll-linked (simple):
```jsx
const { scrollYProgress } = useScroll();
const opacity = useTransform(scrollYProgress, [0, 0.5], [1, 0]);

<motion.div style={{ opacity }} />
```

Bundle optimization with LazyMotion:
```jsx
import { LazyMotion, domAnimation, m } from "motion/react";

// domAnimation = ~15kb (basic features)
// domMax = ~30kb (all features including layout)
<LazyMotion features={domAnimation}>
  <m.div animate={{ opacity: 1 }} />
</LazyMotion>
```

**When to use Motion vs GSAP:**
- Motion: React component animations, layout transitions, gesture-driven interactions, exit
  animations, spring physics on UI elements
- GSAP: Scroll-driven storytelling, pinning, complex timelines with multiple elements, SVG
  morphing, text splitting, anything requiring scrub sync with scroll position

They can coexist. Use Motion for component-level micro-interactions and GSAP + Lenis for the
page-level scroll choreography.

---

## 5. React Three Fiber (R3F)

**Status (2026):** v9 is stable (pairs with React 19). v10 is in alpha with WebGPU support,
new scheduler, and TSL hooks. Use v9 for production.

**Installation:**
```bash
npm install @react-three/fiber @react-three/drei @react-three/postprocessing three
```

**Basic scene in Next.js:**
```jsx
"use client";
import { Canvas } from "@react-three/fiber";
import { OrbitControls, Environment } from "@react-three/drei";

export default function Scene() {
  return (
    <Canvas camera={{ position: [0, 0, 5], fov: 50 }}>
      <ambientLight intensity={0.5} />
      <directionalLight position={[10, 10, 5]} intensity={1} />
      <mesh>
        <boxGeometry args={[1, 1, 1]} />
        <meshStandardMaterial color="#ff6b35" />
      </mesh>
      <OrbitControls enableDamping />
      <Environment preset="city" />
    </Canvas>
  );
}
```

**Scroll-synced 3D (R3F + GSAP + Lenis):**
Use GSAP's ScrollTrigger to drive Three.js uniforms or object transforms:
```jsx
useGSAP(() => {
  gsap.to(meshRef.current.rotation, {
    y: Math.PI * 2,
    scrollTrigger: {
      trigger: ".scroll-section",
      start: "top top",
      end: "bottom bottom",
      scrub: 1,
    },
  });
}, { dependencies: [] });
```

**Key @react-three/drei helpers:**
- `useGLTF` — Load 3D models (GLTF/GLB format)
- `Float` — Auto-floating animation
- `Text3D` / `Text` — 3D and billboard text
- `MeshTransmissionMaterial` — Glass/refraction effects
- `ScrollControls` / `Scroll` — Drei's own scroll system (alternative to GSAP for simpler cases)
- `useProgress` — Loading progress for `<Suspense>` fallbacks

**Performance rules for 3D:**
- Keep polygon count under 100k for smooth scroll experiences
- Use `<Instances>` for repeated geometry
- Compress textures (KTX2 format via `useKTX2`)
- Use `frameloop="demand"` on Canvas if the scene is mostly static
- Lazy-load the entire Canvas component with `next/dynamic`

---

## 6. CSS-Native Scroll Animations

For Tier 1 projects or progressive enhancement. No JS required.

**scroll-timeline (modern browsers):**
```css
@keyframes fade-in {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}

.card {
  animation: fade-in linear both;
  animation-timeline: view();           /* Triggers as element enters viewport */
  animation-range: entry 0% entry 100%; /* Start → end of entry into view */
}
```

**@starting-style (entry animations without JS):**
```css
.dialog[open] {
  opacity: 1;
  transform: scale(1);
  transition: opacity 0.3s, transform 0.3s, display 0.3s allow-discrete;

  @starting-style {
    opacity: 0;
    transform: scale(0.95);
  }
}
```

**Tailwind v4 3D transform utilities:**
```html
<div class="rotate-x-12 rotate-y-6 perspective-800 translate-z-10">
  3D transformed element
</div>
```

---

## 7. Combination Recipes

### Recipe A: Premium Landing Page (Lenis + GSAP + Motion)
- Lenis: Smooth scroll foundation
- GSAP ScrollTrigger: Section pinning, parallax, scrub animations
- Motion: Component-level hover states, modal transitions, layout animations
- CSS: Base transitions for simple hover/focus states

### Recipe B: 3D Product Showcase (Lenis + GSAP + R3F)
- Lenis: Smooth scroll
- GSAP: Drive scroll-to-3D-transform mapping, pin during 3D exploration
- R3F: Render product model, handle rotation/zoom
- Load R3F lazily — only when the 3D section scrolls into range

### Recipe C: Content-Heavy Site with Polish (Lenis + CSS + Motion)
- Lenis: Smooth scroll for editorial feel
- CSS scroll-timeline: Lightweight reveals for cards/images
- Motion: Page transition animations via AnimatePresence + View Transitions API
- No GSAP needed — keep the bundle lean

---

## 8. Animation Performance Rules

1. **Compositor-only properties:** Animate ONLY `transform` and `opacity` wherever possible.
   These bypass layout/paint and run on the GPU.

2. **will-change:** Use sparingly and only on elements about to animate. Remove after animation
   completes. Never apply to more than ~5 elements simultaneously.

3. **Single RAF loop:** When using Lenis + GSAP, let GSAP's ticker be the single
   requestAnimationFrame source. Never run multiple competing RAF loops.

4. **Dynamic imports:** Load GSAP, Lenis, R3F, and Motion dynamically for routes that need them.
   ```js
   const Scene = dynamic(() => import("./Scene"), { ssr: false });
   ```

5. **Debounce ScrollTrigger refresh:** ScrollTrigger already debounces resize (200ms gap). Don't
   add your own resize handlers that call `ScrollTrigger.refresh()` redundantly.

6. **Reduce for a11y:** Always check `prefers-reduced-motion`. Disable scrub, pin, parallax.
   Show content in its final state immediately.
