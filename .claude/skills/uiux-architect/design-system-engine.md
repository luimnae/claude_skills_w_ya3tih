# Design System Engine Reference

How to bootstrap a complete design system from a simple prompt. Every project gets tokens defined
before markup. Last verified: March 2026.

## Table of Contents
1. Typography Architecture
2. Color Systems
3. Spacing & Layout
4. Visual Depth & Texture
5. Accessibility as Architecture
6. Font Pairing Cheat Sheet

---

## 1. Typography Architecture

### Type Scale

Use a fluid scale with `clamp()` for responsive typography without breakpoints:

```css
@theme {
  --text-xs: clamp(0.694rem, 0.05vw + 0.68rem, 0.75rem);
  --text-sm: clamp(0.833rem, 0.09vw + 0.8rem, 0.9rem);
  --text-base: clamp(1rem, 0.15vw + 0.96rem, 1.125rem);
  --text-lg: clamp(1.2rem, 0.25vw + 1.13rem, 1.35rem);
  --text-xl: clamp(1.44rem, 0.4vw + 1.33rem, 1.69rem);
  --text-2xl: clamp(1.728rem, 0.6vw + 1.56rem, 2.11rem);
  --text-3xl: clamp(2.074rem, 0.9vw + 1.83rem, 2.64rem);
  --text-4xl: clamp(2.488rem, 1.3vw + 2.13rem, 3.3rem);
  --text-5xl: clamp(2.986rem, 1.8vw + 2.48rem, 4.12rem);
}
```

This scale uses a ~1.2 ratio (minor third). Adjust ratio for personality:
- 1.125 (major second) — Subtle, editorial, lots of body text
- 1.2 (minor third) — Balanced, most web projects
- 1.25 (major third) — Bold, marketing, fewer text levels
- 1.333 (perfect fourth) — Dramatic, posters, hero-heavy

### Variable Fonts

Prefer variable fonts for performance (one file, all weights) and fine-grained control:

```css
@font-face {
  font-family: "Cabinet Grotesk";
  src: url("/fonts/CabinetGrotesk-Variable.woff2") format("woff2-variations");
  font-weight: 100 900;
  font-display: swap;
  font-style: normal;
}
```

**Font loading strategy:**
1. `font-display: swap` — Show fallback immediately, swap when loaded
2. `size-adjust` on fallback — Minimize layout shift during swap
3. `preload` critical fonts in `<head>`
4. Subset fonts to needed character sets (latin, latin-ext)

```html
<link rel="preload" href="/fonts/CabinetGrotesk-Variable.woff2"
      as="font" type="font/woff2" crossorigin />
```

**Next.js font optimization:**
```tsx
// lib/fonts.ts
import localFont from "next/font/local";

export const display = localFont({
  src: "../public/fonts/CabinetGrotesk-Variable.woff2",
  variable: "--font-display",
  display: "swap",
});

export const body = localFont({
  src: "../public/fonts/Satoshi-Variable.woff2",
  variable: "--font-body",
  display: "swap",
});

// app/layout.tsx
<body className={`${display.variable} ${body.variable} font-body`}>
```

### Font Selection Rules

**DO:**
- Choose fonts with personality that match the project's emotional intent
- Pair a distinctive display font with a clean body font
- Test at actual sizes used in the design (not just specimen pages)
- Check that the font has the weights you need (especially medium/semibold for UI)

**DON'T:**
- Use Inter, Roboto, Arial, Helvetica, or system-ui as primary display font
- Pair two fonts from the same "vibe" (both geometric, both humanist)
- Use more than 2 font families (3 is absolute max, with a mono for code)
- Use decorative/script fonts for body text

### Curated Font Sources (free, high-quality)

**Display fonts:** Cabinet Grotesk, General Sans, Clash Display, Switzer, Zodiak, Boska,
Neue Montreal, Fragment Mono, Space Grotesk (only if not overused in context), Instrument Serif,
Bricolage Grotesque, Syne, Unbounded, Plus Jakarta Sans.

**Body fonts:** Satoshi, Outfit, DM Sans, Source Serif 4, Lora, Literata, Newsreader,
Noto Serif, IBM Plex Sans, Geist (Vercel's font).

**Mono fonts:** JetBrains Mono, Fira Code, Geist Mono, IBM Plex Mono, Fragment Mono.

**Sources:** fonts.google.com, fontsource.org (npm packages), fontshare.com (free commercial),
atipo.es, uncut.wtf.

---

## 2. Color Systems

### OKLCH-First Palette Generation

Tailwind v4 uses OKLCH by default. OKLCH is perceptually uniform — equal lightness values
actually look equally bright (unlike HSL where yellow at 50% looks brighter than blue at 50%).

**OKLCH structure:** `oklch(Lightness Chroma Hue)`
- Lightness: 0 (black) to 1 (white)
- Chroma: 0 (gray) to ~0.4 (maximum saturation, varies by hue)
- Hue: 0-360 degrees (same wheel as HSL)

**Generating a palette from a brand color:**

```css
@theme {
  /* Primary — brand color expanded into a usable scale */
  --color-primary-50:  oklch(0.97 0.02 250);
  --color-primary-100: oklch(0.93 0.04 250);
  --color-primary-200: oklch(0.87 0.07 250);
  --color-primary-300: oklch(0.78 0.11 250);
  --color-primary-400: oklch(0.68 0.14 250);
  --color-primary-500: oklch(0.58 0.16 250);  /* Base brand color */
  --color-primary-600: oklch(0.50 0.14 250);
  --color-primary-700: oklch(0.42 0.12 250);
  --color-primary-800: oklch(0.35 0.09 250);
  --color-primary-900: oklch(0.27 0.07 250);
  --color-primary-950: oklch(0.20 0.05 250);
}
```

**Pattern:** Keep the hue constant. Decrease lightness from 50 → 950. Chroma peaks at 400-500
(most saturated) and decreases at both extremes (light tints and dark shades need less chroma).

### Dark Mode Architecture

Dark mode is NOT just "invert the lightness." It requires intentional mapping:

```css
@theme {
  /* Light mode surfaces */
  --color-surface-0: oklch(1 0 0);        /* Pure white background */
  --color-surface-1: oklch(0.97 0.005 250); /* Subtle tinted surface */
  --color-surface-2: oklch(0.94 0.01 250);  /* Card background */

  /* Dark mode surfaces (set via [data-theme="dark"] or .dark) */
  --color-surface-0-dark: oklch(0.15 0.01 250);  /* NOT pure black — slightly tinted */
  --color-surface-1-dark: oklch(0.19 0.015 250);  /* Elevated surface */
  --color-surface-2-dark: oklch(0.23 0.02 250);   /* Card on dark */
}
```

**Rules for dark mode:**
- Never use pure black (#000) as background — use very dark tinted color (oklch ~0.13-0.17)
- Reduce chroma slightly in dark mode — saturated colors are harsh on dark backgrounds
- Text goes from dark-on-light to light-on-dark, but NOT pure white — use oklch(0.93 ...)
- Elevation in dark mode = lighter surfaces (opposite of light mode shadows)
- Primary color may need lightness adjustment to maintain contrast

### Semantic Colors

Always define semantic tokens that map to the palette:

```css
@theme {
  --color-success: oklch(0.62 0.17 145);
  --color-warning: oklch(0.75 0.15 75);
  --color-error: oklch(0.58 0.2 25);
  --color-info: oklch(0.62 0.15 250);
}
```

### Contrast Requirements

WCAG 2.2 AA minimum contrast ratios:
- Normal text: 4.5:1
- Large text (18px+ or 14px bold+): 3:1
- UI components and graphical objects: 3:1

Use `oklch()` lightness values to quickly check: if the difference between text lightness and
background lightness is > 0.45, you're likely meeting AA for normal text.

---

## 3. Spacing & Layout

### Base Grid System

Use 4px base unit (Tailwind v4 default: 0.25rem = 4px):

```
4px  (1)  — Tight gaps, icon padding
8px  (2)  — Default inline spacing
12px (3)  — Small component padding
16px (4)  — Standard padding, paragraph gaps
24px (6)  — Section internal spacing
32px (8)  — Component separation
48px (12) — Section padding
64px (16) — Major section gaps
96px (24) — Hero breathing room
128px (32) — Maximum spacing
```

### Fluid Spacing

Use `clamp()` for spacing that scales with viewport:

```css
.section-padding {
  padding-block: clamp(3rem, 5vw + 1rem, 8rem);
}
```

### Container Queries over Media Queries

Tailwind v4 has native container query support. Prefer container queries for component-level
responsiveness:

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row @lg:grid @lg:grid-cols-3 gap-4 @md:gap-6 @lg:gap-8">
    <!-- Layout responds to parent container, not viewport -->
  </div>
</div>
```

### CSS Grid Patterns

**Auto-fill responsive grid (no breakpoints):**
```html
<div class="grid grid-cols-[repeat(auto-fill,minmax(min(100%,300px),1fr))] gap-6">
  <!-- Cards auto-flow into available columns -->
</div>
```

**Subgrid for aligned nested grids:**
```css
.parent { display: grid; grid-template-columns: repeat(3, 1fr); }
.child { display: grid; grid-template-columns: subgrid; grid-column: span 3; }
```

---

## 4. Visual Depth & Texture

### Layered Shadow System

Single-layer `box-shadow` looks flat. Layer multiple shadows for realistic depth:

```css
@theme {
  --shadow-sm: 0 1px 2px oklch(0 0 0 / 0.04), 0 1px 1px oklch(0 0 0 / 0.06);
  --shadow-md: 0 2px 4px oklch(0 0 0 / 0.04), 0 4px 8px oklch(0 0 0 / 0.06),
               0 1px 2px oklch(0 0 0 / 0.04);
  --shadow-lg: 0 4px 8px oklch(0 0 0 / 0.03), 0 8px 16px oklch(0 0 0 / 0.06),
               0 16px 32px oklch(0 0 0 / 0.06), 0 2px 4px oklch(0 0 0 / 0.03);
  --shadow-xl: 0 8px 16px oklch(0 0 0 / 0.04), 0 16px 32px oklch(0 0 0 / 0.08),
               0 32px 64px oklch(0 0 0 / 0.08), 0 4px 8px oklch(0 0 0 / 0.03);
}
```

**Colored shadows (match the element's dominant color):**
```css
.card-blue { box-shadow: 0 8px 32px oklch(0.55 0.15 250 / 0.25); }
```

### Glassmorphism / Frosted Glass

```css
.glass {
  background: oklch(1 0 0 / 0.1);
  backdrop-filter: blur(20px) saturate(1.5);
  border: 1px solid oklch(1 0 0 / 0.15);
}
```

### Noise & Grain Textures

Add subtle noise to flat surfaces for organic feel:

```css
.textured {
  position: relative;
}
.textured::after {
  content: "";
  position: absolute;
  inset: 0;
  background-image: url("data:image/svg+xml,..."); /* tiny noise SVG */
  opacity: 0.03;
  pointer-events: none;
  mix-blend-mode: overlay;
}
```

Or generate noise with CSS (no external asset):
```css
.noise {
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
}
```

### Gradient Mesh

For complex, multi-color gradient backgrounds:
```css
.gradient-mesh {
  background-color: oklch(0.25 0.05 280);
  background-image:
    radial-gradient(at 20% 80%, oklch(0.45 0.2 320) 0%, transparent 50%),
    radial-gradient(at 80% 20%, oklch(0.5 0.18 250) 0%, transparent 50%),
    radial-gradient(at 50% 50%, oklch(0.35 0.15 200) 0%, transparent 60%);
}
```

---

## 5. Accessibility as Architecture

Accessibility is not a checklist — it's a structural decision made before any visual design.

### Non-Negotiable Requirements

- **WCAG 2.2 AA** as minimum. Target AAA for text contrast.
- **Semantic HTML** — Use `<button>` not `<div onClick>`. Use `<nav>`, `<main>`, `<article>`.
- **Focus management** — Every interactive element must have a visible focus indicator.
  Never `outline: none` without a replacement.
- **Skip links** — First focusable element should be "Skip to main content."
- **prefers-reduced-motion** — Disable scroll hijacking, parallax, auto-playing animations.
  Show content in final state.
- **prefers-color-scheme** — Respect system dark mode preference.
- **prefers-contrast** — Increase contrast when requested.

### Focus Indicator Pattern

```css
:focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
  border-radius: 4px;
}
```

### Reduced Motion Pattern

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### React Hook for Reduced Motion

```tsx
export function useReducedMotion() {
  const [reduced, setReduced] = useState(false);
  useEffect(() => {
    const mql = window.matchMedia("(prefers-reduced-motion: reduce)");
    setReduced(mql.matches);
    const handler = (e: MediaQueryListEvent) => setReduced(e.matches);
    mql.addEventListener("change", handler);
    return () => mql.removeEventListener("change", handler);
  }, []);
  return reduced;
}
```

### ARIA for Custom Components

Custom components need ARIA roles, states, and properties. shadcn/ui handles this
automatically for its components. For custom work:
- Dropdowns: `role="listbox"`, `aria-expanded`, `aria-activedescendant`
- Tabs: `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected`
- Modals: `role="dialog"`, `aria-modal="true"`, trap focus, return focus on close
- Toasts: `role="status"`, `aria-live="polite"`

---

## 6. Font Pairing Cheat Sheet

Quick-reference pairs organized by project personality:

| Personality | Display Font | Body Font | Vibe |
|------------|-------------|-----------|------|
| Modern Tech | Cabinet Grotesk | Satoshi | Clean, confident, contemporary |
| Editorial Luxury | Instrument Serif | DM Sans | Refined, magazine-quality |
| Bold Creative | Clash Display | General Sans | Energetic, agency, portfolio |
| Warm Humanist | Bricolage Grotesque | Source Serif 4 | Approachable, trustworthy |
| Brutalist Raw | Fragment Mono | Space Mono | Stark, technical, unapologetic |
| Playful Modern | Syne | Outfit | Fun, startup, consumer app |
| Classic Premium | Zodiak | Literata | Timeless, high-end, editorial |
| Minimal Geometric | Switzer | Geist | Precise, systematic, developer tools |
