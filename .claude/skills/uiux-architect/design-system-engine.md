# Design System Engine Reference
Define tokens BEFORE any markup. Every design decision flows from these. Last verified: March 2026.

## Contents
1. Typography Architecture
2. Color Systems
3. Spacing & Layout
4. Visual Depth & Texture
5. Accessibility
6. Font Pairing Cheat Sheet

---

## 1. Typography Architecture

**Fluid type scale with `clamp()` — no breakpoints needed:**
```css
@theme {
  --text-xs:   clamp(0.694rem, 0.05vw + 0.68rem, 0.75rem);
  --text-sm:   clamp(0.833rem, 0.09vw + 0.8rem, 0.9rem);
  --text-base: clamp(1rem, 0.15vw + 0.96rem, 1.125rem);
  --text-lg:   clamp(1.2rem, 0.25vw + 1.13rem, 1.35rem);
  --text-xl:   clamp(1.44rem, 0.4vw + 1.33rem, 1.69rem);
  --text-2xl:  clamp(1.728rem, 0.6vw + 1.56rem, 2.11rem);
  --text-3xl:  clamp(2.074rem, 0.9vw + 1.83rem, 2.64rem);
  --text-4xl:  clamp(2.488rem, 1.3vw + 2.13rem, 3.3rem);
  --text-5xl:  clamp(2.986rem, 1.8vw + 2.48rem, 4.12rem);
}
```

Scale ratio by personality: `1.125` editorial · `1.2` balanced (default) · `1.25` bold marketing · `1.333` dramatic/posters

**Variable fonts (one file, all weights):**
```css
@font-face {
  font-family: "Cabinet Grotesk";
  src: url("/fonts/CabinetGrotesk-Variable.woff2") format("woff2-variations");
  font-weight: 100 900;
  font-display: swap;
}
```

**Font loading strategy:** `font-display: swap` → `size-adjust` on fallback to minimize CLS → `preload` critical fonts → subset to latin

**Next.js:**
```tsx
// lib/fonts.ts
import localFont from "next/font/local";
export const display = localFont({ src: "../public/fonts/CabinetGrotesk-Variable.woff2", variable: "--font-display", display: "swap" });
export const body = localFont({ src: "../public/fonts/Satoshi-Variable.woff2", variable: "--font-body", display: "swap" });

// app/layout.tsx
<body className={`${display.variable} ${body.variable} font-body`}>
```

**Rules:** Choose fonts with personality. Pair distinctive display with clean body. Test at actual sizes. Max 2 families (3 with mono for code). Never decorative/script for body text. Never Inter/Roboto/Arial/Helvetica as primary display.

**Font Sources (free, high-quality):**
- Display: Cabinet Grotesk · General Sans · Clash Display · Switzer · Zodiak · Boska · Neue Montreal · Fragment Mono · Instrument Serif · Bricolage Grotesque · Syne · Unbounded
- Body: Satoshi · Outfit · DM Sans · Source Serif 4 · Lora · Geist
- Mono: JetBrains Mono · Fira Code · Geist Mono · IBM Plex Mono
- Sources: fontsource.org · fontshare.com · atipo.es · uncut.wtf

---

## 2. Color Systems

**OKLCH-first (Tailwind v4 default). Perceptually uniform — equal lightness values look equally bright.**

`oklch(Lightness Chroma Hue)` — L: 0-1 · C: 0-~0.4 · H: 0-360

**Palette from a brand color:**
```css
@theme {
  --color-primary-50:  oklch(0.97 0.02 250);
  --color-primary-100: oklch(0.93 0.04 250);
  --color-primary-200: oklch(0.87 0.07 250);
  --color-primary-300: oklch(0.78 0.11 250);
  --color-primary-400: oklch(0.68 0.14 250);
  --color-primary-500: oklch(0.58 0.16 250); /* Base */
  --color-primary-600: oklch(0.50 0.14 250);
  --color-primary-700: oklch(0.42 0.12 250);
  --color-primary-800: oklch(0.35 0.09 250);
  --color-primary-900: oklch(0.27 0.07 250);
  --color-primary-950: oklch(0.20 0.05 250);
}
```
Pattern: keep hue constant · chroma peaks at 400-500 · decreases at both extremes.

**Dark mode surfaces:**
```css
@theme {
  --color-surface-0: oklch(1 0 0);           /* Light: pure white */
  --color-surface-1: oklch(0.97 0.005 250);
  --color-surface-2: oklch(0.94 0.01 250);
  --color-surface-0-dark: oklch(0.15 0.01 250); /* Dark: NOT pure black */
  --color-surface-1-dark: oklch(0.19 0.015 250);
  --color-surface-2-dark: oklch(0.23 0.02 250);
}
```
Dark mode rules: never `#000` background · reduce chroma slightly · text not pure white (oklch ~0.93) · elevation = lighter surfaces (opposite of light mode).

**Semantic tokens:**
```css
@theme {
  --color-success: oklch(0.62 0.17 145);
  --color-warning: oklch(0.75 0.15 75);
  --color-error:   oklch(0.58 0.2 25);
  --color-info:    oklch(0.62 0.15 250);
}
```

**Contrast (WCAG 2.2 AA):** Normal text 4.5:1 · Large text 3:1 · UI components 3:1. Quick check: lightness diff > 0.45 ≈ AA pass.

---

## 3. Spacing & Layout

**4px base unit (Tailwind v4 default: 0.25rem):**
```
4px(1) tight · 8px(2) inline · 12px(3) small padding · 16px(4) standard
24px(6) section internal · 32px(8) component gap · 48px(12) section padding
64px(16) major gaps · 96px(24) hero breathing room · 128px(32) max
```

**Fluid spacing:**
```css
.section-padding { padding-block: clamp(3rem, 5vw + 1rem, 8rem); }
```

**Container queries over media queries (Tailwind v4 native):**
```html
<div class="@container">
  <div class="flex flex-col @md:flex-row @lg:grid @lg:grid-cols-3 gap-4 @md:gap-6">
    <!-- Responds to parent container, not viewport -->
  </div>
</div>
```

**Auto-fill responsive grid (no breakpoints):**
```html
<div class="grid grid-cols-[repeat(auto-fill,minmax(min(100%,300px),1fr))] gap-6">
```

**Subgrid:**
```css
.parent { display: grid; grid-template-columns: repeat(3, 1fr); }
.child { display: grid; grid-template-columns: subgrid; grid-column: span 3; }
```

---

## 4. Visual Depth & Texture

**Layered shadows (realistic depth):**
```css
@theme {
  --shadow-sm: 0 1px 2px oklch(0 0 0 / 0.04), 0 1px 1px oklch(0 0 0 / 0.06);
  --shadow-md: 0 2px 4px oklch(0 0 0 / 0.04), 0 4px 8px oklch(0 0 0 / 0.06), 0 1px 2px oklch(0 0 0 / 0.04);
  --shadow-lg: 0 4px 8px oklch(0 0 0 / 0.03), 0 8px 16px oklch(0 0 0 / 0.06), 0 16px 32px oklch(0 0 0 / 0.06);
  --shadow-xl: 0 8px 16px oklch(0 0 0 / 0.04), 0 16px 32px oklch(0 0 0 / 0.08), 0 32px 64px oklch(0 0 0 / 0.08);
}
/* Colored shadow — match element's dominant color */
.card-blue { box-shadow: 0 8px 32px oklch(0.55 0.15 250 / 0.25); }
```

**Glassmorphism:**
```css
.glass {
  background: oklch(1 0 0 / 0.1);
  backdrop-filter: blur(20px) saturate(1.5);
  border: 1px solid oklch(1 0 0 / 0.15);
}
```

**Noise/grain texture (CSS, no external asset):**
```css
.noise {
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
}
```

**Gradient mesh:**
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

## 5. Accessibility

**Non-negotiable:** WCAG 2.2 AA minimum · semantic HTML · visible focus on every interactive element · skip links · `prefers-reduced-motion` · `prefers-color-scheme` · `prefers-contrast`

**Focus indicator:**
```css
:focus-visible {
  outline: 2px solid var(--color-primary-500);
  outline-offset: 2px;
  border-radius: 4px;
}
```

**Reduced motion CSS:**
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

**React hook:**
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

**ARIA for custom components:** Dropdowns: `role="listbox"` + `aria-expanded` · Tabs: `role="tablist/tab/tabpanel"` + `aria-selected` · Modals: `role="dialog"` + `aria-modal` + focus trap + return focus · Toasts: `role="status"` + `aria-live="polite"`

---

## 6. Font Pairing Cheat Sheet

| Personality | Display | Body | Vibe |
|---|---|---|---|
| Modern Tech | Cabinet Grotesk | Satoshi | Clean, confident |
| Editorial Luxury | Instrument Serif | DM Sans | Magazine-quality |
| Bold Creative | Clash Display | General Sans | Agency, portfolio |
| Warm Humanist | Bricolage Grotesque | Source Serif 4 | Approachable |
| Brutalist Raw | Fragment Mono | Space Mono | Stark, technical |
| Playful Modern | Syne | Outfit | Startup, consumer |
| Classic Premium | Zodiak | Literata | Timeless, high-end |
| Minimal Geometric | Switzer | Geist | Developer tools |
