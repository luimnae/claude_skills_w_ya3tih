# Anti-Patterns Reference
Load for design reviews, audits, or "why does this look generic/AI-generated?" Last verified: March 2026.

---

## 1. AI Slop Giveaways

**Purple gradient on white.** Most common AI cliché. If you reach for purple, ask: does this project actually call for it?

**Inter/Roboto/Arial.** These are defaults, not choices. Every project deserves a font chosen for personality.

**Uniform padding.** When every element has `p-6` or `p-8` the design feels mechanical. Real design has rhythm.

**Cookie-cutter card grids.** Three identical cards (icon + heading + paragraph + button). Break the grid. Make one larger. Vary density.

**Overuse of rounded corners.** `rounded-2xl` on everything = toy aesthetic unless intentional. Mix rounding levels.

**Generic hero.** Big heading + subtitle + 2 buttons + gradient blob = placeholder, not design. A real hero has a point of view.

**Rainbow palette.** 5+ saturated colors with equal weight = visual noise. Strong palette: 1-2 dominant, 1 accent, rest neutrals.

### Escape Slop
1. **Commit to ONE bold aesthetic.** Typography-led, texture-led, space-led, or motion-led. Pick one, push it hard.
2. **Study 3-5 real sites** on Awwwards/Godly before designing. Internalize their decision-making.
3. **Design with constraints.** "Make it look good" = generic. "1970s Swiss magazine for Gen Z" = memorable.
4. **Typography first.** Strong type with simple color beats strong color with weak type.
5. **Whitespace is a feature.** Premium design uses space as aggressively as content.

---

## 2. Architecture Anti-Patterns

**`"use client"` on everything.** Every boundary increases the JS bundle. Fix: extract interactive parts as small Client Components. A page with 20 components should have 3-5 `"use client"` boundaries.

**Prop drilling through 10 layers.** Fix: Zustand for shared client state. RSC for server data — fetch where you render.

**One giant Client Component for animations.** Fix: small focused wrappers — `<ScrollReveal>`, `<ParallaxLayer>`, `<MagneticButton>`. Page layout stays Server Component.

**Importing full libraries.**
```js
// BAD
import * as gsap from "gsap";
// GOOD
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
```

**No error boundaries.** Wrap R3F `<Canvas>`, dynamic imports, and data-dependent visualizations.

**Mixing animation systems chaotically.** Establish ownership:
- CSS → simple hover/focus state transitions
- Motion → component lifecycle (enter, exit, layout, gestures)
- GSAP → scroll-driven (everything ScrollTrigger-related)

**`tailwind.config.js` in a v4 project.** v4 uses CSS-first config via `@theme`. The config file is gone.

---

## 3. UX Anti-Patterns

**Scroll hijacking without escape.** Lenis smooths, doesn't hijack. For pinned sections: always provide a clear visual cue content continues.

**Animations that block interaction.** Nav must be interactive immediately. Interactive elements must be usable even if animations haven't finished. Use `pointer-events: auto` on interactive elements.

**Missing reduced motion fallbacks.** Not optional — affects vestibular disorders, motion sensitivity, epilepsy. When `prefers-reduced-motion: reduce`: disable parallax/scrub/pin, disable Lenis, show content in final state.

**Auto-playing video/audio.** Always require user interaction. If hero background video, mute it and provide a pause button.

**Infinite scroll without position memory.** Save scroll position before nav. Restore on back. Use `scrollRestoration` or manual tracking.

**Form UX failures:** clearing on error, errors only on submit, no loading state, no success confirmation. Fix: inline validation on blur, preserve input, disable button + spinner on submit.

---

## 4. Animation Anti-Patterns

**Everything bounces.** `elastic`/`bounce` on every animation = toy. Default: `power2.out` or `cubic-bezier(0.16, 1, 0.3, 1)`. Reserve elastic for one or two intentional moments.

**Entrance animation on every element.** 40 identical fade-up staggers = boring. Animate sections, not individual elements. Or animate only the 3-4 most important moments.

**Duration too long.** Over 600ms for UI interactions feels sluggish. Micro-interactions: 100-200ms. Section transitions: 300-500ms. Page transitions: 200-400ms. 600ms+ only for dramatic intentional moments.

**Competing animations.** Multiple animations fighting for attention simultaneously. Only ONE starring animation per viewport. Supporting elements: subtle motion only.

**No cleanup.** GSAP ScrollTriggers not killed on unmount = memory leaks and ghost animations. Always `useGSAP()` with scope. Destroy Lenis in cleanup.

---

## 5. The "Looks Professional" Trap

Everything is technically correct — clean code, good a11y, good perf — but completely forgettable.

**Signs you're in it:**
- Swap the logo and it could be any company
- Nothing a user would screenshot or share
- Template structure: hero → features → testimonials → CTA → footer
- Every section has the same visual weight

**Escape:**
- Find the ONE thing about this brand that's different. Design around that.
- Break ONE convention intentionally. Overlapping elements. Unexpected scroll. A section that's "wrong" in an interesting way.
- Remove something. The design has too many sections. Cut the weakest one.
- Add ONE moment of delight — a hover effect, a scroll reveal, a typographic treatment that makes people pause.
- Ask: "Would I remember this site tomorrow?" If no, you're in the trap.
