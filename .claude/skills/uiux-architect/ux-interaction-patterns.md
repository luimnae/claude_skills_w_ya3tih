# UX Interaction Patterns Reference

Apple-level interaction design, page transitions, micro-interactions, and gesture systems.
Last verified: March 2026.

## Table of Contents
1. Apple-Level Scroll Patterns
2. Page & View Transitions
3. Micro-Interactions
4. Gesture Systems
5. Loading & State Patterns
6. Responsive Interaction Design

---

## 1. Apple-Level Scroll Patterns

Apple's product pages define the gold standard for scroll-driven storytelling. Here are the
core patterns deconstructed:

### Sticky Section with Progressive Reveal

Pin a section, then reveal content elements sequentially as user scrolls:
```js
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: ".product-section",
    start: "top top",
    end: "+=300%",
    pin: true,
    scrub: 1,
  },
});

tl.from(".product-image", { scale: 0.8, opacity: 0, duration: 1 })
  .from(".product-title", { y: 60, opacity: 0, duration: 0.5 }, "-=0.3")
  .from(".product-description", { y: 40, opacity: 0, duration: 0.5 }, "-=0.2")
  .from(".product-specs", { y: 30, opacity: 0, stagger: 0.1 }, "-=0.1");
```

### Parallax Depth Layers

Multiple layers moving at different scroll speeds create depth:
```js
gsap.to(".layer-far", { yPercent: -10, scrollTrigger: { trigger: ".hero", scrub: true } });
gsap.to(".layer-mid", { yPercent: -30, scrollTrigger: { trigger: ".hero", scrub: true } });
gsap.to(".layer-near", { yPercent: -60, scrollTrigger: { trigger: ".hero", scrub: true } });
```

**Rule of thumb:** Far layers move 10-15% of scroll, mid layers 25-35%, near layers 50-70%.
Foreground text should stay pinned or move minimally.

### Zoom-Through Transition

Image or 3D object scales up as you scroll, creating a "flying into" effect:
```js
gsap.to(".zoom-image", {
  scale: 15,
  opacity: 0,
  scrollTrigger: {
    trigger: ".zoom-section",
    start: "top top",
    end: "+=150%",
    pin: true,
    scrub: 1,
  },
});
```

### Text Split & Reveal

Split text into lines/words/chars and animate individually. Use GSAP's SplitText plugin
or implement manually:
```js
// Manual word splitting
const text = element.textContent;
element.innerHTML = text.split(" ").map(word =>
  `<span class="word"><span class="word-inner">${word}</span></span>`
).join(" ");

gsap.from(".word-inner", {
  yPercent: 100,
  opacity: 0,
  stagger: 0.05,
  duration: 0.8,
  ease: "power3.out",
  scrollTrigger: {
    trigger: element,
    start: "top 80%",
  },
});
```

### Horizontal Scroll Section

Vertical scroll drives horizontal movement (common for portfolio/gallery):
```js
const sections = gsap.utils.toArray(".horizontal-panel");

gsap.to(sections, {
  xPercent: -100 * (sections.length - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".horizontal-container",
    pin: true,
    scrub: 1,
    snap: 1 / (sections.length - 1),
    end: () => "+=" + document.querySelector(".horizontal-container").scrollWidth,
  },
});
```

### Color/Theme Transition on Scroll

Background color shifts as you scroll between sections:
```js
const sections = gsap.utils.toArray(".color-section");
sections.forEach((section) => {
  const bg = section.dataset.bg;
  const text = section.dataset.text;

  ScrollTrigger.create({
    trigger: section,
    start: "top 60%",
    end: "bottom 40%",
    onEnter: () => gsap.to("body", { backgroundColor: bg, color: text, duration: 0.5 }),
    onEnterBack: () => gsap.to("body", { backgroundColor: bg, color: text, duration: 0.5 }),
  });
});
```

---

## 2. Page & View Transitions

### View Transitions API (React 19.2 + Next.js 16)

The View Transitions API enables smooth animations between page navigations. Next.js 16
supports this natively:

```jsx
// Link with transition type
<Link href="/about" transitionTypes={["slide"]}>About</Link>

// CSS for the transition
::view-transition-old(root) {
  animation: slide-out 0.3s ease-in-out;
}
::view-transition-new(root) {
  animation: slide-in 0.3s ease-in-out;
}

@keyframes slide-out {
  to { transform: translateX(-100%); opacity: 0; }
}
@keyframes slide-in {
  from { transform: translateX(100%); opacity: 0; }
}
```

### Shared Element Transitions

Elements that persist across routes animate smoothly between positions:
```css
.product-image {
  view-transition-name: product-hero;
}

/* On the detail page, same view-transition-name on the larger image */
.product-detail-image {
  view-transition-name: product-hero;
}
```

### AnimatePresence for Route Transitions (Motion)

For SPA-style transitions without the View Transitions API:
```jsx
"use client";
import { AnimatePresence, motion } from "motion/react";
import { usePathname } from "next/navigation";

export default function PageTransition({ children }) {
  const pathname = usePathname();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={pathname}
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

### Magnetic Button

Button that subtly follows the cursor when hovering nearby:
```jsx
function MagneticButton({ children }) {
  const ref = useRef(null);

  const handleMouseMove = (e) => {
    const { left, top, width, height } = ref.current.getBoundingClientRect();
    const x = (e.clientX - left - width / 2) * 0.3;
    const y = (e.clientY - top - height / 2) * 0.3;
    gsap.to(ref.current, { x, y, duration: 0.3, ease: "power2.out" });
  };

  const handleMouseLeave = () => {
    gsap.to(ref.current, { x: 0, y: 0, duration: 0.5, ease: "elastic.out(1, 0.3)" });
  };

  return (
    <button ref={ref} onMouseMove={handleMouseMove} onMouseLeave={handleMouseLeave}>
      {children}
    </button>
  );
}
```

### Tilt Card (3D perspective on hover)

```jsx
function TiltCard({ children }) {
  const ref = useRef(null);

  const handleMouseMove = (e) => {
    const { left, top, width, height } = ref.current.getBoundingClientRect();
    const rotateX = ((e.clientY - top) / height - 0.5) * -15;
    const rotateY = ((e.clientX - left) / width - 0.5) * 15;
    gsap.to(ref.current, {
      rotateX, rotateY,
      transformPerspective: 800,
      duration: 0.4,
      ease: "power2.out",
    });
  };

  const handleMouseLeave = () => {
    gsap.to(ref.current, { rotateX: 0, rotateY: 0, duration: 0.6, ease: "power2.out" });
  };

  return (
    <div ref={ref} onMouseMove={handleMouseMove} onMouseLeave={handleMouseLeave}
         style={{ transformStyle: "preserve-3d" }}>
      {children}
    </div>
  );
}
```

### Glow Effect on Hover

Radial gradient follows the cursor position:
```css
.glow-card {
  position: relative;
  overflow: hidden;
}
.glow-card::before {
  content: "";
  position: absolute;
  inset: 0;
  background: radial-gradient(
    600px circle at var(--mouse-x, 50%) var(--mouse-y, 50%),
    oklch(0.7 0.15 250 / 0.15),
    transparent 40%
  );
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.3s;
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

### Button Press Feedback

```jsx
<motion.button
  whileTap={{ scale: 0.97 }}
  whileHover={{ scale: 1.02 }}
  transition={{ type: "spring", stiffness: 400, damping: 17 }}
>
  Click me
</motion.button>
```

### Staggered List Entry

```jsx
const containerVariants = {
  hidden: {},
  visible: { transition: { staggerChildren: 0.08 } },
};
const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.4, ease: [0.16, 1, 0.3, 1] } },
};

<motion.ul variants={containerVariants} initial="hidden" animate="visible">
  {items.map((item) => (
    <motion.li key={item.id} variants={itemVariants}>{item.name}</motion.li>
  ))}
</motion.ul>
```

---

## 4. Gesture Systems

### Drag with Spring Physics (Motion)

```jsx
<motion.div
  drag
  dragConstraints={{ left: -100, right: 100, top: -50, bottom: 50 }}
  dragElastic={0.2}
  dragTransition={{ bounceStiffness: 300, bounceDamping: 20 }}
  whileDrag={{ scale: 1.05, cursor: "grabbing" }}
/>
```

### Swipe Actions (Mobile)

```jsx
const x = useMotionValue(0);
const opacity = useTransform(x, [-200, 0, 200], [0.5, 1, 0.5]);

<motion.div
  style={{ x, opacity }}
  drag="x"
  dragConstraints={{ left: 0, right: 0 }}
  onDragEnd={(_, info) => {
    if (info.offset.x > 100) onSwipeRight();
    if (info.offset.x < -100) onSwipeLeft();
  }}
/>
```

### Custom Cursor

```jsx
function CustomCursor() {
  const cursorRef = useRef(null);

  useEffect(() => {
    const move = (e) => {
      gsap.to(cursorRef.current, {
        x: e.clientX,
        y: e.clientY,
        duration: 0.5,
        ease: "power3.out",
      });
    };
    window.addEventListener("mousemove", move);
    return () => window.removeEventListener("mousemove", move);
  }, []);

  return (
    <div ref={cursorRef} className="pointer-events-none fixed top-0 left-0 z-50
         h-5 w-5 -translate-x-1/2 -translate-y-1/2 rounded-full bg-primary-500
         mix-blend-difference" />
  );
}
```

---

## 5. Loading & State Patterns

### Skeleton Loading (Tailwind v4)

```html
<div class="animate-pulse space-y-4">
  <div class="h-4 w-3/4 rounded bg-surface-2"></div>
  <div class="h-4 w-1/2 rounded bg-surface-2"></div>
  <div class="h-32 rounded-lg bg-surface-2"></div>
</div>
```

### Shimmer Effect

```css
.shimmer {
  background: linear-gradient(
    90deg,
    oklch(0.95 0 0) 25%,
    oklch(0.90 0 0) 50%,
    oklch(0.95 0 0) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

### Progress Indicator with Motion

```jsx
<motion.div
  className="h-1 bg-primary-500 origin-left"
  initial={{ scaleX: 0 }}
  animate={{ scaleX: progress }}
  transition={{ duration: 0.3, ease: "easeOut" }}
/>
```

---

## 6. Responsive Interaction Design

### Touch vs Pointer Adaptation

```css
/* Hover effects only for pointer devices */
@media (hover: hover) and (pointer: fine) {
  .card:hover { transform: translateY(-4px); }
}

/* Larger touch targets for touch devices */
@media (pointer: coarse) {
  .button { min-height: 44px; min-width: 44px; }
}
```

### Animation Scaling by Viewport

Reduce animation complexity on mobile for performance:
```js
const isMobile = window.matchMedia("(max-width: 768px)").matches;

gsap.from(".hero-element", {
  y: isMobile ? 30 : 100,           // Less movement on mobile
  duration: isMobile ? 0.5 : 0.8,   // Shorter duration
  stagger: isMobile ? 0.05 : 0.1,   // Tighter stagger
});
```

### Progressive Enhancement

Start with CSS-only animations. Layer on JS for capable devices:
```js
if (!window.matchMedia("(prefers-reduced-motion: reduce)").matches
    && "IntersectionObserver" in window) {
  // Initialize GSAP scroll animations
  initScrollAnimations();
} else {
  // Content is already visible via CSS defaults
}
```
