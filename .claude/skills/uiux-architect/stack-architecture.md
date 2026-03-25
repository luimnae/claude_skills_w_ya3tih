# Stack Architecture Reference

Framework selection, state management, styling, component ecosystem, and project structure.
Last verified: March 2026.

## Table of Contents
1. Framework Selection Matrix
2. State Management
3. Tailwind CSS v4
4. shadcn/ui v4
5. Project Structure Conventions
6. Rendering Strategy Decision Tree

---

## 1. Framework Selection Matrix

### Next.js 16.2 (Default Choice)

**Use when:** Most projects. SSR/SSG/ISR needed. SEO matters. Full-stack capabilities.
Complex routing. Deployed on Vercel or self-hosted.

**Key features (16.x):**
- **App Router** is the default and recommended router. Pages Router is maintenance-mode.
- **React Server Components (RSC)** — fetch data on server, zero JS sent to client for server components
- **Cache Components** — `"use cache"` directive for explicit, opt-in caching (replaces v14/v15 implicit caching). Works with PPR for instant navigation.
- **Partial Prerendering (PPR)** — Static shell served from edge, dynamic content streamed in
- **proxy.ts** — Replaces `middleware.ts`. Runs on Node.js (NOT edge). Use for rewrites, redirects, auth checks at the network boundary
- **Turbopack** — Stable as default bundler. 5-10x faster Fast Refresh, 2-5x faster builds
- **View Transitions** — React 19.2 feature, supported in App Router. `<Link transitionTypes={['slide']}>` for route-based transitions
- **React Compiler** — Stable, opt-in. Auto-memoization, no manual `useMemo`/`useCallback` needed when enabled
- **Server Functions** — Renamed from "Server Actions". Handle forms and mutations without API routes

**Setup:**
```bash
npx create-next-app@latest my-app
# Defaults: App Router, TypeScript, Tailwind CSS, ESLint, Turbopack
```

**Critical patterns:**
- Default to Server Components. Only add `"use client"` when you need useState, useEffect, event handlers, or browser APIs
- Use `"use cache"` on pages/components/functions that benefit from caching — it's fully opt-in now
- Heavy animation components (GSAP, Lenis, R3F) MUST be Client Components
- Wrap Client Components in Server Components for data fetching

### Astro 5

**Use when:** Content-heavy sites (blogs, docs, marketing). Maximum performance with minimal JS.
Static-first with islands of interactivity.

**Key strengths:** Zero JS by default, content collections, island architecture, works with
React/Vue/Svelte components, outstanding Lighthouse scores.

**Not ideal for:** Highly interactive apps, real-time features, complex client-side state.

### Vite + React

**Use when:** SPA with no SSR requirement. Internal tools. Prototypes. Projects where you want
full control without framework opinions.

**Setup:** `npm create vite@latest my-app -- --template react-ts`

### Remix / React Router 7

**Use when:** Data-heavy apps with complex form handling. Projects that need progressive
enhancement. Teams that prefer web standards over framework abstractions.

---

## 2. State Management

### Decision Tree

```
Is it server data (API responses, DB queries)?
  → TanStack Query (React Query) for client-side cache/sync
  → Or just RSC + Server Functions for simple cases

Is it UI state (modals, tabs, sidebar open)?
  → useState / useReducer (component-local)

Is it shared client state across many components?
  → Zustand (default recommendation — simple, performant, no boilerplate)

Is it derived/computed state from multiple atoms?
  → Jotai (atomic model, excellent for derived state)

Is it form state?
  → React Hook Form + Zod for validation
```

### Zustand (Default Recommendation)

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

Why Zustand over Redux/Context:
- No providers needed (works outside React tree too)
- No boilerplate (actions and state in one place)
- Built-in selectors for render optimization
- Tiny bundle (~1kb)
- Works with RSC architecture (store stays on client)

---

## 3. Tailwind CSS v4

**Released January 2025. Major rewrite. Not backwards-compatible with v3 config.**

### What Changed

**CSS-first configuration:** No more `tailwind.config.js`. Everything in CSS via `@theme`:
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

**Single-line import:** `@import "tailwindcss"` replaces the three `@tailwind` directives.

**OKLCH colors by default:** More perceptually uniform than RGB/HSL. Better for generating
consistent palettes. All default colors are OKLCH.

**Native container queries (no plugin):**
```html
<div class="@container">
  <div class="grid grid-cols-1 @sm:grid-cols-2 @lg:grid-cols-4">
    <!-- Responds to container width, not viewport -->
  </div>
</div>
```

**3D transform utilities:**
```html
<div class="rotate-x-12 rotate-y-6 perspective-800 translate-z-10">
  Transforms in 3D space via utility classes
</div>
```

**Dynamic spacing:** Every multiple of the 0.25rem base is available. `mt-13`, `p-7`, `gap-18`
all work without arbitrary values.

**Cascade layers:** Uses real CSS `@layer` rules. Specificity issues from v3 are eliminated.

**color-mix() for opacity:** `bg-primary-500/50` uses native `color-mix()` under the hood.

**@starting-style support:** For entry animations without JS.

### Key Migration Notes (v3 → v4)
- `bg-gradient-to-r` → `bg-linear-to-r`
- `flex-shrink-0` → `shrink-0` (already canonical in v3 but now enforced)
- `overflow-ellipsis` → `text-ellipsis`
- Run `npx @tailwindcss/upgrade` for automatic class renaming (handles ~90%)
- `@apply` still works but is discouraged — prefer explicit CSS properties

---

## 4. shadcn/ui v4

**Status (March 2026):** CLI v4, presets, registry:base, Radix or Base UI primitives,
namespaced registries, `shadcn/skills` for AI agents.

### Key Concepts

**Not a component library — a code distribution platform.** Components are copied into your
project as source code. You own them. No `node_modules` black box.

**CLI v4 commands:**
```bash
npx shadcn@latest init          # Scaffold project (Next.js, Vite, Astro, etc.)
npx shadcn@latest add button    # Add a component
npx shadcn@latest add block-login-01  # Add a full page block
npx shadcn@latest add @registry/component  # Add from community registry
npx shadcn@latest add button --dry-run     # Preview what will be added
npx shadcn@latest add button --diff        # Check for updates
```

**Presets:** Entire design system as a single code string. Colors, fonts, radius, icon library.
Build custom presets on `shadcn/create`, preview live, grab the code.

**Primitive choice:** During `init`, choose between:
- **Radix UI** — Full-featured, excellent accessibility, more opinionated
- **Base UI** — Unstyled, lightweight, from the MUI team, zero-style overhead

**Component styles (5 options):**
- Vega (New York) — Classic shadcn look
- Nova — Compact, reduced padding
- Maia — Soft, rounded, generous spacing
- Lyra — Boxy, sharp, pairs with mono fonts
- Mira — Compact, made for dense interfaces

**Registry ecosystem (community registries):**
```bash
npx shadcn@latest add @aceternity/hero      # Aceternity UI
npx shadcn@latest add @magicui/shimmer       # Magic UI
npx shadcn@latest add @shadcnblocks/hero125  # ShadcnBlocks (pro)
```

Notable registries: Aceternity UI (animated components), Magic UI (animated primitives),
React Bits (animation primitives), Kokonut UI (interaction-driven marketing components).

**shadcn/skills:** Gives coding agents (Claude Code, Cursor) context about shadcn components,
APIs, CLI commands, and registry workflows. Install for better agent output.

### Usage Pattern in Projects

```bash
# 1. Init with preset
npx shadcn@latest init --preset "your-preset-code"

# 2. Add components as needed
npx shadcn@latest add button card dialog sheet tabs

# 3. Add blocks for full pages
npx shadcn@latest add block-dashboard-01

# 4. Customize the source in components/ui/
# 5. Override theme in CSS via @theme
```

---

## 5. Project Structure Conventions

### Next.js 16 App Router Structure

```
src/
├── app/
│   ├── layout.tsx              # Root layout (fonts, providers, metadata)
│   ├── page.tsx                # Home page
│   ├── globals.css             # Tailwind import + @theme
│   ├── (marketing)/            # Route group — no URL segment
│   │   ├── page.tsx
│   │   └── pricing/page.tsx
│   ├── dashboard/
│   │   ├── layout.tsx          # Dashboard-specific layout
│   │   └── page.tsx
│   └── api/                    # API routes (if needed beyond Server Functions)
├── components/
│   ├── ui/                     # shadcn/ui components (auto-generated)
│   ├── sections/               # Page sections (Hero, Features, Pricing, etc.)
│   ├── animations/             # Reusable animation wrappers
│   │   ├── smooth-scroll-provider.tsx
│   │   ├── scroll-reveal.tsx
│   │   └── magnetic-button.tsx
│   └── layouts/                # Shared layout components
├── lib/
│   ├── utils.ts                # cn() helper, formatters
│   ├── motion-variants.ts      # Reusable Motion animation variants
│   └── fonts.ts                # Font configuration
├── hooks/
│   ├── use-media-query.ts
│   └── use-reduced-motion.ts
├── stores/                     # Zustand stores
│   └── app-store.ts
└── types/
    └── index.ts
```

**Key conventions:**
- `(parentheses)` for route groups — organize without affecting URL
- Colocate page-specific components inside route directories
- Keep `components/ui/` sacred — only shadcn auto-generated code
- `components/sections/` for page-level building blocks
- `components/animations/` for reusable motion wrappers
- One Zustand store per domain, not one global store

---

## 6. Rendering Strategy Decision Tree

```
Is this page content-heavy with rare updates?
  → Static Generation (default in Next.js 16 with no dynamic code)
  → Add "use cache" for explicit caching

Does this page have personalized or real-time data?
  → Server Component with streaming + Suspense
  → Use PPR: static shell + dynamic islands

Does this page need SEO + frequent data updates?
  → ISR (Incremental Static Regeneration) via revalidate option

Is this a fully interactive SPA-like experience?
  → Client Component with TanStack Query for data
  → Still wrap in Server Component layout for initial HTML

Is this a dashboard with WebSocket/real-time data?
  → Client Component for the data layer
  → Server Components for the layout shell
```

**General rule:** Start with Server Components. Wrap interactive islands in `"use client"`.
Push `"use client"` boundaries as deep into the tree as possible.
