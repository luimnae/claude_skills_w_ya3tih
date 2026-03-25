# Animation Systems Reference
Read fully before implementing Tier 2+. Last verified: March 2026.

## Contents
1. GSAP 3.14+
2. Lenis 1.3.x
3. Lenis + GSAP Integration (Default for Tier 3)
4. Motion 12.x
5. R3F v9
6. CSS-Native Scroll
7. Combination Recipes
8. Performance Rules

---

## 1. GSAP 3.14+

Free for all uses including commercial (post-Webflow acquisition).

```bash
npm install gsap @gsap/react
```

**useGSAP hook — always use in React (auto-cleanup on unmount):**
```jsx
import { useGSAP } from "@gsap/react";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

function HeroSection() {
  const containerRef = useRef(null);

  useGSAP(() => {
    gsap.from(".hero-title", { y: 100, opacity: 0, duration: 1, ease: "power3.out" });
    gsap.to(".hero-image", {
      yPercent: -20,
      scrollTrigger: { trigger: ".hero-section", start: "top top", end: "bottom top", scrub: true },
    });
  }, { scope: containerRef });

  return <div ref={containerRef}>...</div>;
}
```

**Core ScrollTrigger patterns:**

Pin + scrub timeline:
```js
gsap.timeline({
  scrollTrigger: { trigger: ".section", start: "top top", end: "+=200%", pin: true, scrub: 1 },
})
.to(".title", { opacity: 1, y: 0 })
.to(".image", { scale: 1.2 })
.to(".text", { opacity: 1 });
```

Snap to sections:
```js
ScrollTrigger.create({
  trigger: ".sections-container",
  start: "top top", end: "bottom bottom",
  snap: 1 / (numSections - 1),
});
```

Horizontal scroll via vertical:
```js
gsap.to(".horizontal-wrapper", {
  xPercent: -100 * (panels.length - 1) / panels.length,
  ease: "none",
  scrollTrigger: {
    trigger: ".horizontal-container", pin: true, scrub: 1,
    end: () => "+=" + document.querySelector(".horizontal-wrapper").scrollWidth,
  },
});
```

Staggered scroll reveal:
```js
gsap.from(".card", {
  y: 60, opacity: 0, stagger: 0.15, duration: 0.8, ease: "power2.out",
  scrollTrigger: { trigger: ".cards-grid", start: "top 80%" },
});
```

Animate CSS variable (multi-property sync):
```js
gsap.to(".element", { "--progress": 1, scrollTrigger: { trigger: ".element", scrub: true } });
```
```css
.element {
  transform: scale(calc(1 + var(--progress) * 0.5));
  opacity: calc(0.3 + var(--progress) * 0.7);
}
```

**prefers-reduced-motion:**
```js
const prefersReduced = window.matchMedia("(prefers-reduced-motion: reduce)");
if (prefersReduced.matches) {
  gsap.set(".hero-title", { opacity: 1, y: 0 });
} else {
  gsap.from(".hero-title", { opacity: 0, y: 100, duration: 1 });
}
```

---

## 2. Lenis 1.3.x

Package: `lenis` (not `@studio-freight/lenis` — deprecated). Latest: 1.3.20.

```bash
npm install lenis
```

**Standalone (no GSAP):**
```js
import Lenis from "lenis";
import "lenis/dist/lenis.css";
const lenis = new Lenis({ autoRaf: true });
```

**Required CSS (global styles):**
```css
html.lenis { height: auto; }
.lenis.lenis-smooth { scroll-behavior: auto !important; }
.lenis.lenis-smooth [data-lenis-prevent] { overscroll-behavior: contain; }
.lenis.lenis-stopped { overflow: hidden; }
.lenis.lenis-scrolling iframe { pointer-events: none; }
```

**Nested scrollable elements:** `data-lenis-prevent` | `data-lenis-prevent-wheel` | `data-lenis-prevent-touch`

**Next.js App Router:**
```jsx
// app/lenis-provider.jsx
"use client";
import { ReactLenis } from "lenis/react";
export default function LenisProvider({ children }) {
  return <ReactLenis root options={{ autoRaf: true }}>{children}</ReactLenis>;
}
```

**Key options:** `lerp: 0.1` (smoothness 0.05-0.5) · `wheelMultiplier: 1` · `smoothTouch: false` (keep false for mobile) · `infinite: false`

---

## 3. Lenis + GSAP Integration (Recommended Default for Tier 3)

Lenis = smooth scroll physics. GSAP ScrollTrigger = animation triggering. Sync via GSAP's ticker.

**Why Lenis over ScrollSmoother:** No DOM restructuring needed. Better with position:fixed elements.

```js
import Lenis from "lenis";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

const lenis = new Lenis(); // No autoRaf — GSAP drives it
lenis.on("scroll", ScrollTrigger.update);
gsap.ticker.add((time) => { lenis.raf(time * 1000); });
gsap.ticker.lagSmoothing(0);
```

**React/Next.js full setup:**
```jsx
"use client";
import { useEffect } from "react";
import Lenis from "lenis";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

export default function SmoothScrollProvider({ children }) {
  useEffect(() => {
    const lenis = new Lenis();
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

Rebranded from "Framer Motion". Import path changed. React + Vue + vanilla JS.

```bash
npm install motion
```

```jsx
// CRITICAL — do NOT import from framer-motion
import { motion, AnimatePresence, useScroll, useTransform } from "motion/react";
```

**Core patterns:**
```jsx
// Layout animations
<motion.div layout layoutId="card">...</motion.div>

// Exit animations
<AnimatePresence mode="wait">
  {show && (
    <motion.div key="modal"
      initial={{ opacity: 0, scale: 0.95 }}
      animate={{ opacity: 1, scale: 1 }}
      exit={{ opacity: 0, scale: 0.95 }}
      transition={{ type: "spring", damping: 25, stiffness: 300 }}
    />
  )}
</AnimatePresence>

// Scroll-linked
const { scrollYProgress } = useScroll();
const opacity = useTransform(scrollYProgress, [0, 0.5], [1, 0]);
<motion.div style={{ opacity }} />
```

**Bundle optimization:**
```jsx
import { LazyMotion, domAnimation, m } from "motion/react";
// domAnimation ~15kb · domMax ~30kb
<LazyMotion features={domAnimation}>
  <m.div animate={{ opacity: 1 }} />
</LazyMotion>
```

**Motion vs GSAP:**
- Motion → component animations, layout transitions, gestures, exit animations, spring physics
- GSAP → scroll-driven storytelling, pinning, complex timelines, SVG morphing, text splitting, scrub sync

They coexist. Motion for micro-interactions, GSAP+Lenis for page-level scroll choreography.

---

## 5. R3F v9

v9 = stable (React 19). v10 = alpha (WebGPU). Use v9 for production.

```bash
npm install @react-three/fiber @react-three/drei @react-three/postprocessing three
```

**Basic scene:**
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

**Scroll-synced 3D:**
```jsx
useGSAP(() => {
  gsap.to(meshRef.current.rotation, {
    y: Math.PI * 2,
    scrollTrigger: { trigger: ".scroll-section", start: "top top", end: "bottom bottom", scrub: 1 },
  });
}, { dependencies: [] });
```

**Key drei helpers:** `useGLTF` · `Float` · `Text3D` / `Text` · `MeshTransmissionMaterial` (glass) · `ScrollControls` · `useProgress`

**Performance:** polygon count < 100k · `<Instances>` for repeated geometry · KTX2 textures · `frameloop="demand"` for static scenes · lazy-load with `next/dynamic`

---

## 6. CSS-Native Scroll

For Tier 1 or progressive enhancement. Zero JS.

```css
/* scroll-timeline */
@keyframes fade-in {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}
.card {
  animation: fade-in linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}

/* @starting-style (entry animations without JS) */
.dialog[open] {
  opacity: 1; transform: scale(1);
  transition: opacity 0.3s, transform 0.3s, display 0.3s allow-discrete;
  @starting-style { opacity: 0; transform: scale(0.95); }
}
```

**Tailwind v4 3D utilities:**
```html
<div class="rotate-x-12 rotate-y-6 perspective-800 translate-z-10">3D element</div>
```

---

## 7. Combination Recipes

**Recipe A: Premium Landing Page** → Lenis (scroll) + GSAP ScrollTrigger (pinning/scrub) + Motion (component hovers/modals) + CSS (base hover/focus)

**Recipe B: 3D Product Showcase** → Lenis + GSAP (scroll-to-3D mapping, pinning) + R3F (model, rotation/zoom). Load R3F lazily when 3D section enters range.

**Recipe C: Content-Heavy Site** → Lenis + CSS scroll-timeline (card reveals) + Motion (page transitions via AnimatePresence + View Transitions). No GSAP — lean bundle.

---

## 8. Performance Rules

1. **Compositor-only:** Animate `transform` and `opacity` only. These run on GPU, skip layout/paint.
2. **`will-change`:** Use sparingly, only on elements about to animate, max ~5 simultaneously. Remove after animation.
3. **Single RAF loop:** GSAP ticker drives Lenis. Never two competing RAF loops.
4. **Dynamic imports:** Load GSAP, Lenis, R3F, Motion dynamically for routes that need them.
   ```js
   const Scene = dynamic(() => import("./Scene"), { ssr: false });
   ```
5. **No redundant refresh:** ScrollTrigger debounces resize (200ms). Don't add your own `ScrollTrigger.refresh()` on resize.
6. **Reduced motion:** Always check `prefers-reduced-motion`. Disable scrub, pin, parallax. Show final state immediately.
