# UX Interaction Patterns Reference
Apple-level interactions, page transitions, micro-interactions, gestures. Last verified: March 2026.

## Contents
1. Apple-Level Scroll Patterns
2. Page & View Transitions
3. Micro-Interactions
4. Gesture Systems
5. Loading & State Patterns
6. Responsive Interaction

---

## 1. Apple-Level Scroll Patterns

**Sticky section with progressive reveal:**
```js
const tl = gsap.timeline({
  scrollTrigger: { trigger: ".product-section", start: "top top", end: "+=300%", pin: true, scrub: 1 },
});
tl.from(".product-image", { scale: 0.8, opacity: 0, duration: 1 })
  .from(".product-title", { y: 60, opacity: 0, duration: 0.5 }, "-=0.3")
  .from(".product-description", { y: 40, opacity: 0, duration: 0.5 }, "-=0.2")
  .from(".product-specs", { y: 30, opacity: 0, stagger: 0.1 }, "-=0.1");
```

**Parallax depth layers:**
```js
gsap.to(".layer-far",  { yPercent: -10, scrollTrigger: { trigger: ".hero", scrub: true } });
gsap.to(".layer-mid",  { yPercent: -30, scrollTrigger: { trigger: ".hero", scrub: true } });
gsap.to(".layer-near", { yPercent: -60, scrollTrigger: { trigger: ".hero", scrub: true } });
// Rule: far 10-15% · mid 25-35% · near 50-70%. Foreground text stays pinned.
```

**Zoom-through transition:**
```js
gsap.to(".zoom-image", {
  scale: 15, opacity: 0,
  scrollTrigger: { trigger: ".zoom-section", start: "top top", end: "+=150%", pin: true, scrub: 1 },
});
```

**Text split & reveal:**
```js
// Manual word split
const text = element.textContent;
element.innerHTML = text.split(" ").map(w => `<span class="word"><span class="word-inner">${w}</span></span>`).join(" ");

gsap.from(".word-inner", {
  yPercent: 100, opacity: 0, stagger: 0.05, duration: 0.8, ease: "power3.out",
  scrollTrigger: { trigger: element, start: "top 80%" },
});
```

**Horizontal scroll (vertical drives horizontal):**
```js
const sections = gsap.utils.toArray(".horizontal-panel");
gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".horizontal-container", pin: true, scrub: 1,
    snap: 1 / (sections.length - 1),
    end: () => "+=" + document.querySelector(".horizontal-container").scrollWidth,
  },
});
```

**Color/theme transition on scroll:**
```js
gsap.utils.toArray(".color-section").forEach((section) => {
  ScrollTrigger.create({
    trigger: section, start: "top 60%", end: "bottom 40%",
    onEnter: () => gsap.to("body", { backgroundColor: section.dataset.bg, color: section.dataset.text, duration: 0.5 }),
    onEnterBack: () => gsap.to("body", { backgroundColor: section.dataset.bg, color: section.dataset.text, duration: 0.5 }),
  });
});
```

---

## 2. Page & View Transitions

**View Transitions API (Next.js 16 + React 19.2):**
```jsx
<Link href="/about" transitionTypes={["slide"]}>About</Link>
```
```css
::view-transition-old(root) { animation: slide-out 0.3s ease-in-out; }
::view-transition-new(root) { animation: slide-in 0.3s ease-in-out; }
@keyframes slide-out { to { transform: translateX(-100%); opacity: 0; } }
@keyframes slide-in { from { transform: translateX(100%); opacity: 0; } }
```

**Shared element transitions:**
```css
.product-image       { view-transition-name: product-hero; }
.product-detail-image { view-transition-name: product-hero; }
/* Browser animates between positions automatically */
```

**AnimatePresence route transitions (Motion):**
```jsx
"use client";
import { AnimatePresence, motion } from "motion/react";
import { usePathname } from "next/navigation";

export default function PageTransition({ children }) {
  const pathname = usePathname();
  return (
    <AnimatePresence mode="wait">
      <motion.div key={pathname}
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -20 }}
        transition={{ duration: 0.3, ease: [0.16, 1, 0.3, 1] }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}
```

---

## 3. Micro-Interactions

**Magnetic button:**
```jsx
function MagneticButton({ children }) {
  const ref = useRef(null);
  const handleMouseMove = (e) => {
    const { left, top, width, height } = ref.current.getBoundingClientRect();
    gsap.to(ref.current, { x: (e.clientX - left - width / 2) * 0.3, y: (e.clientY - top - height / 2) * 0.3, duration: 0.3, ease: "power2.out" });
  };
  const handleMouseLeave = () => gsap.to(ref.current, { x: 0, y: 0, duration: 0.5, ease: "elastic.out(1, 0.3)" });
  return <button ref={ref} onMouseMove={handleMouseMove} onMouseLeave={handleMouseLeave}>{children}</button>;
}
```

**Tilt card (3D hover):**
```jsx
function TiltCard({ children }) {
  const ref = useRef(null);
  const handleMouseMove = (e) => {
    const { left, top, width, height } = ref.current.getBoundingClientRect();
    gsap.to(ref.current, {
      rotateX: ((e.clientY - top) / height - 0.5) * -15,
      rotateY: ((e.clientX - left) / width - 0.5) * 15,
      transformPerspective: 800, duration: 0.4, ease: "power2.out",
    });
  };
  const handleMouseLeave = () => gsap.to(ref.current, { rotateX: 0, rotateY: 0, duration: 0.6, ease: "power2.out" });
  return <div ref={ref} onMouseMove={handleMouseMove} onMouseLeave={handleMouseLeave} style={{ transformStyle: "preserve-3d" }}>{children}</div>;
}
```

**Glow on hover (cursor-following radial gradient):**
```css
.glow-card { position: relative; overflow: hidden; }
.glow-card::before {
  content: ""; position: absolute; inset: 0; pointer-events: none;
  background: radial-gradient(600px circle at var(--mouse-x, 50%) var(--mouse-y, 50%), oklch(0.7 0.15 250 / 0.15), transparent 40%);
  opacity: 0; transition: opacity 0.3s;
}
.glow-card:hover::before { opacity: 1; }
```
```js
card.addEventListener("mousemove", (e) => {
  const rect = card.getBoundingClientRect();
  card.style.setProperty("--mouse-x", `${e.clientX - rect.left}px`);
  card.style.setProperty("--mouse-y", `${e.clientY - rect.top}px`);
});
```

**Button press:**
```jsx
<motion.button whileTap={{ scale: 0.97 }} whileHover={{ scale: 1.02 }} transition={{ type: "spring", stiffness: 400, damping: 17 }}>
  Click me
</motion.button>
```

**Staggered list entry:**
```jsx
const containerVariants = { hidden: {}, visible: { transition: { staggerChildren: 0.08 } } };
const itemVariants = { hidden: { opacity: 0, y: 20 }, visible: { opacity: 1, y: 0, transition: { duration: 0.4, ease: [0.16, 1, 0.3, 1] } } };

<motion.ul variants={containerVariants} initial="hidden" animate="visible">
  {items.map((item) => <motion.li key={item.id} variants={itemVariants}>{item.name}</motion.li>)}
</motion.ul>
```

---

## 4. Gesture Systems

**Drag with spring physics:**
```jsx
<motion.div
  drag
  dragConstraints={{ left: -100, right: 100, top: -50, bottom: 50 }}
  dragElastic={0.2}
  dragTransition={{ bounceStiffness: 300, bounceDamping: 20 }}
  whileDrag={{ scale: 1.05, cursor: "grabbing" }}
/>
```

**Swipe actions (mobile):**
```jsx
const x = useMotionValue(0);
const opacity = useTransform(x, [-200, 0, 200], [0.5, 1, 0.5]);
<motion.div style={{ x, opacity }} drag="x" dragConstraints={{ left: 0, right: 0 }}
  onDragEnd={(_, info) => {
    if (info.offset.x > 100) onSwipeRight();
    if (info.offset.x < -100) onSwipeLeft();
  }}
/>
```

**Custom cursor:**
```jsx
function CustomCursor() {
  const cursorRef = useRef(null);
  useEffect(() => {
    const move = (e) => gsap.to(cursorRef.current, { x: e.clientX, y: e.clientY, duration: 0.5, ease: "power3.out" });
    window.addEventListener("mousemove", move);
    return () => window.removeEventListener("mousemove", move);
  }, []);
  return <div ref={cursorRef} className="pointer-events-none fixed top-0 left-0 z-50 h-5 w-5 -translate-x-1/2 -translate-y-1/2 rounded-full bg-primary-500 mix-blend-difference" />;
}
```

---

## 5. Loading & State Patterns

**Skeleton (Tailwind v4):**
```html
<div class="animate-pulse space-y-4">
  <div class="h-4 w-3/4 rounded bg-surface-2"></div>
  <div class="h-4 w-1/2 rounded bg-surface-2"></div>
  <div class="h-32 rounded-lg bg-surface-2"></div>
</div>
```

**Shimmer:**
```css
.shimmer {
  background: linear-gradient(90deg, oklch(0.95 0 0) 25%, oklch(0.90 0 0) 50%, oklch(0.95 0 0) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }
```

**Progress bar:**
```jsx
<motion.div className="h-1 bg-primary-500 origin-left" initial={{ scaleX: 0 }} animate={{ scaleX: progress }} transition={{ duration: 0.3, ease: "easeOut" }} />
```

---

## 6. Responsive Interaction

**Hover only for pointer devices:**
```css
@media (hover: hover) and (pointer: fine) {
  .card:hover { transform: translateY(-4px); }
}
@media (pointer: coarse) {
  .button { min-height: 44px; min-width: 44px; }
}
```

**Reduce animation complexity on mobile:**
```js
const isMobile = window.matchMedia("(max-width: 768px)").matches;
gsap.from(".hero-element", {
  y: isMobile ? 30 : 100,
  duration: isMobile ? 0.5 : 0.8,
  stagger: isMobile ? 0.05 : 0.1,
});
```

**Progressive enhancement:**
```js
if (!window.matchMedia("(prefers-reduced-motion: reduce)").matches && "IntersectionObserver" in window) {
  initScrollAnimations();
}
// Otherwise content is already visible via CSS defaults
```
