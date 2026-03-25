# Self-Improvement Protocol

This file defines how the uiux-architect skill evolves and gets better with every use.

---

## 1. Pre-Build Version Check

Before starting any project, if web search or internet access is available, verify current
versions of the core stack. The reference files contain truth as of March 2026 — anything
newer should override them.

**Check these:**
- `gsap` — npm latest version, any new plugins or API changes
- `lenis` — npm latest version, any breaking changes
- `motion` (framer-motion) — npm latest version, new features
- `@react-three/fiber` — npm latest version, React compatibility
- `next` — latest stable version, new features
- `tailwindcss` — latest stable version, new utilities
- `shadcn` CLI — latest version, new components or registries

**How to check (in Claude Code):**
```bash
npm view gsap version
npm view lenis version
npm view motion version
npm view @react-three/fiber version
npm view next version
npm view tailwindcss version
```

If a version has changed significantly from what's documented in the references, note the
discrepancy and adapt your implementation accordingly.

---

## 2. Post-Build Retrospective

After completing any build, run this internal audit before delivering to the user:

### Stack Audit
- Did I choose the right framework? Would a simpler/different choice have been better?
- Did I import any library I didn't actually use?
- Is there dead code or unused CSS?
- Could any Client Component be made into a Server Component?

### Animation Audit
- Does every animation serve a purpose (guide attention, communicate state, create delight)?
- Is there an animation that feels gratuitous or distracting?
- Is there a moment that feels static where animation would help?
- Did I handle `prefers-reduced-motion`?
- Did I clean up all GSAP instances and Lenis on unmount?

### Design Audit
- Would this stand out on Awwwards/Godly, or does it look like every other AI-generated site?
- Is there ONE thing a user would remember about this design?
- Did I fall into any anti-pattern from `references/anti-patterns.md`?
- Is the typography choice genuinely good, or did I reach for a default?
- Does the color palette have hierarchy (dominant, secondary, accent)?

### Performance Audit
- Would this score 90+ on Lighthouse Performance?
- Are heavy libraries dynamically imported?
- Are images optimized with proper sizes/formats?
- Is the total JS bundle within budget?

---

## 3. Learning Accumulation

When a user provides corrections or feedback during a build:

1. **Identify the category:** Is this a design preference, a technical correction, a
   UX improvement, or a performance issue?

2. **Determine if it's generalizable:** Does this correction apply only to this specific
   project, or does it reveal a gap in the skill's knowledge?

3. **If generalizable:** Note which reference file should be updated and what information
   should be added. Examples:
   - "The Lenis + Next.js 16 App Router setup requires X" → update animation-systems.md
   - "This animation feels too slow on mobile" → update performance-engineering.md
   - "This design looks too much like every other site" → update anti-patterns.md

4. **If project-specific:** Don't modify reference files. Adapt for this project only.

---

## 4. Pattern Extraction

After a particularly successful build (user expresses satisfaction, design is exceptional),
extract reusable patterns:

### Animation Patterns
- What scroll choreography worked well?
- What easing curves felt right for this context?
- What combination of libraries produced the smoothest result?

### Design Patterns
- What font pairing worked well?
- What color system was effective?
- What layout approach was distinctive?

### Architecture Patterns
- What component structure was clean and maintainable?
- What state management approach was simplest?
- What rendering strategy balanced performance and interactivity?

These patterns should mentally inform future builds, creating a flywheel of improvement.

---

## 5. Edge Case Documentation

When encountering technical edge cases during builds, document them internally:

**Common edge cases to watch for:**
- Lenis smooth scroll conflicting with modal scroll lock
- GSAP ScrollTrigger positions wrong after dynamic content loads (need `ScrollTrigger.refresh()`)
- R3F Canvas resizing issues in flex containers
- View Transitions API not working with certain animation libraries
- `position: fixed` elements broken by ancestor `transform` or `will-change: transform`
- Tailwind v4 `@theme` variables not available in JS (only CSS)
- shadcn/ui components needing modification for animation compatibility
- Next.js App Router hydration mismatches with animation initial states

When you encounter one of these, remember the solution for future builds.

---

## 6. Feedback Collection

After delivering a build, if appropriate, ask:

"How does this feel? Anything you'd change about the visual direction, the interactions,
or the code structure?"

Use the response to calibrate future decisions:
- If they say "needs more animation" → next time, default to a higher animation tier
- If they say "too busy" → next time, exercise more restraint
- If they say "looks generic" → next time, make bolder aesthetic choices
- If they say "code is hard to follow" → next time, prioritize readability

The goal is convergence toward the user's ideal output with fewer iterations over time.
