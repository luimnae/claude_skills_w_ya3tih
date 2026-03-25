# Stack Architecture Reference
Framework selection, state, styling, components, project structure. Last verified: March 2026.

## Contents
1. Framework Selection
2. State Management
3. Tailwind CSS v4
4. shadcn/ui v4
5. Project Structure
6. Rendering Strategy

---

## 1. Framework Selection

**Next.js 16.2 — Default for most projects.** SSR/SSG/ISR, SEO, full-stack, complex routing.

Key features: App Router (default, Pages Router is maintenance-mode) · RSC (fetch on server, zero client JS) · `"use cache"` directive (opt-in caching, replaces v14/v15 implicit) · PPR (static shell from edge, dynamic streamed) · `proxy.ts` (replaces `middleware.ts`, runs on Node.js) · Turbopack (stable default, 5-10x faster HMR) · View Transitions (React 19.2, `<Link transitionTypes={['slide']}>`) · React Compiler (stable, opt-in, auto-memoization) · Server Functions (replaces Server Actions)

```bash
npx create-next-app@latest my-app
# Defaults: App Router, TypeScript, Tailwind, ESLint, Turbopack
```

Critical patterns: default Server Components · `"use client"` only for useState/useEffect/events/browser APIs · GSAP/Lenis/R3F MUST be Client Components · wrap Client Components in Server Components for data fetching

**Astro 5 — Content-heavy sites** (blogs, docs, marketing). Zero JS by default, island architecture, outstanding Lighthouse. Not for highly interactive apps or real-time features.

**Vite + React — SPAs with no SSR.** Internal tools, prototypes, full control.
```bash
npm create vite@latest my-app -- --template react-ts
```

**Remix / React Router 7 — Data-heavy apps** with complex form handling, progressive enhancement, web standards preference.

---

## 2. State Management

```
Server data (API/DB)?
  → TanStack Query for client-side cache/sync
  → Or RSC + Server Functions for simple cases

UI state (modals, tabs, sidebar)?
  → useState / useReducer (component-local)

Shared client state across many components?
  → Zustand (simple, performant, no boilerplate)

Derived/computed state from multiple atoms?
  → Jotai

Form state?
  → React Hook Form + Zod
```

**Zustand:**
```tsx
import { create } from "zustand";

interface AppStore {
  sidebarOpen: boolean;
  toggleSidebar: () => void;
  theme: "light" | "dark";
  setTheme: (theme: "light" | "dark") => void;
}

export const useAppStore = create<AppStore>((set) => ({
  sidebarOpen: false,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  theme: "light",
  setTheme: (theme) => set({ theme }),
}));
```

Why Zustand: no providers · no boilerplate · built-in selectors · ~1kb · works outside React tree

---

## 3. Tailwind CSS v4

Major rewrite (Jan 2025). NOT backwards-compatible with v3.

**CSS-first config — no `tailwind.config.js`:**
```css
@import "tailwindcss";

@theme {
  --font-display: "Cabinet Grotesk", sans-serif;
  --font-body: "Satoshi", sans-serif;
  --color-primary-500: oklch(0.65 0.15 250);
  --color-surface: oklch(0.98 0.005 250);
  --breakpoint-3xl: 1920px;
  --ease-smooth: cubic-bezier(0.16, 1, 0.3, 1);
}
```

**What changed:**
- Single import: `@import "tailwindcss"` (replaces 3 `@tailwind` directives)
- OKLCH colors by default
- Native container queries: `@container` / `@sm:` / `@lg:` (no plugin)
- 3D transforms: `rotate-x-12 rotate-y-6 perspective-800 translate-z-10`
- Dynamic spacing: `mt-13`, `p-7`, `gap-18` all work
- Real CSS `@layer` rules (no more specificity issues)
- `color-mix()` for opacity under the hood
- `@starting-style` support

**v3 → v4 gotchas:** `bg-gradient-to-r` → `bg-linear-to-r` · `flex-shrink-0` → `shrink-0` · `overflow-ellipsis` → `text-ellipsis` · run `npx @tailwindcss/upgrade` (handles ~90%)

---

## 4. shadcn/ui v4

Not a component library — a code distribution platform. Components copied into your project as source. You own them.

**CLI:**
```bash
npx shadcn@latest init
npx shadcn@latest add button card dialog sheet tabs
npx shadcn@latest add block-dashboard-01        # Full page blocks
npx shadcn@latest add @aceternity/hero          # Community registries
npx shadcn@latest add button --diff             # Check for updates
```

**Presets:** Entire design system as a single code string. Build on `shadcn/create`.

**Primitive choice:** Radix UI (full-featured, excellent a11y) or Base UI (unstyled, lightweight, from MUI team)

**5 styles:** Vega (classic) · Nova (compact) · Maia (soft/rounded) · Lyra (boxy/sharp) · Mira (dense interfaces)

**Community registries:** Aceternity UI (animated) · Magic UI (animated primitives) · React Bits · Kokonut UI · ShadcnBlocks

**shadcn/skills:** Install for better agent output in Claude Code / Cursor.

**Usage:**
```bash
npx shadcn@latest init --preset "your-preset-code"
npx shadcn@latest add button card dialog
# Customize source in components/ui/
# Override theme in CSS via @theme
```

---

## 5. Project Structure (Next.js 16 App Router)

```
src/
├── app/
│   ├── layout.tsx              # Root layout (fonts, providers, metadata)
│   ├── page.tsx                # Home
│   ├── globals.css             # @import "tailwindcss" + @theme
│   ├── (marketing)/            # Route group — no URL segment
│   │   ├── page.tsx
│   │   └── pricing/page.tsx
│   ├── dashboard/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   └── api/
├── components/
│   ├── ui/                     # shadcn/ui (auto-generated — don't touch)
│   ├── sections/               # Page sections: Hero, Features, Pricing
│   ├── animations/             # Reusable wrappers
│   │   ├── smooth-scroll-provider.tsx
│   │   ├── scroll-reveal.tsx
│   │   └── magnetic-button.tsx
│   └── layouts/
├── lib/
│   ├── utils.ts                # cn() helper, formatters
│   ├── motion-variants.ts      # Reusable Motion variants
│   └── fonts.ts
├── hooks/
│   ├── use-media-query.ts
│   └── use-reduced-motion.ts
├── stores/                     # Zustand (one store per domain)
│   └── app-store.ts
└── types/
    └── index.ts
```

Key conventions: `(parentheses)` for route groups · colocate page-specific components in route dirs · `ui/` is sacred (shadcn only) · one Zustand store per domain

---

## 6. Rendering Strategy

```
Content-heavy, rare updates?
  → Static Generation (default in Next.js 16)
  → Add "use cache" for explicit caching

Personalized or real-time data?
  → Server Component + streaming + Suspense
  → PPR: static shell + dynamic islands

SEO + frequent data updates?
  → ISR via revalidate option

Fully interactive SPA-like?
  → Client Component + TanStack Query
  → Wrap in Server Component layout for initial HTML

Dashboard with WebSocket/real-time?
  → Client Component for data layer
  → Server Components for layout shell
```

**Rule:** Start Server Components. Push `"use client"` boundaries as deep as possible.
