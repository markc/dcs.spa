---
version: alpha
name: DCS Behavior
description: Behavioral specification for the DCS (Dual Carousel Sidebars) reference implementation — interaction model, state machines, persistence, motion, responsive rules, and accessibility. Complements DESIGN.md (visual tokens) with the "what happens when" layer. Authoritative source is docs/base.js + docs/site.js; this file documents observable behavior.
motion:
  duration-fast: 150ms
  duration-base: 200ms
  duration-slow: 300ms
  duration-medium: 400ms
  duration-long: 500ms
  ease-out: cubic-bezier(0.33, 1, 0.68, 1)
  ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1)
  reduced-motion: 0.01ms
breakpoints:
  tablet: 600px
  desktop-narrow: 900px
  desktop: 1200px
  sidebar-pin: 1280px
persistence:
  key: base-state
  storage: localStorage
  encoding: JSON
  fields:
    theme: "'light' | 'dark' | undefined"
    scheme: "'default' | 'crimson' | 'stone' | 'forest' | 'sunset' | 'mono'"
    carousel: "'slide' | 'fade'"
    leftOpen: boolean
    rightOpen: boolean
    leftPinned: boolean
    rightPinned: boolean
    leftPanel: integer
    rightPanel: integer
    sidebarWidthLeft: "integer, 10–100, step 5"
    sidebarWidthRight: "integer, 10–100, step 5"
    treeExpanded: "array of branch indices"
interactions:
  theme-toggle:
    trigger: "click on [.theme-toggle]"
    effect: "swap html.dark ↔ html.light class; update sun/moon icon via Lucide; persist theme"
    duration: "{motion.duration-slow} on body background-color and color"
  theme-set:
    trigger: "click on [data-theme='light'|'dark']"
    effect: "set html class explicitly; persist theme; mark active button"
  scheme-set:
    trigger: "click on [data-scheme='<name>']"
    effect: "remove all scheme-* classes from html; add scheme-<name> (unless 'default'); persist scheme; mark active swatch"
  sidebar-toggle:
    trigger: "click on [.menu-toggle][data-sidebar='left'|'right']"
    effect: "toggle .open on sidebar; closing also removes .pinned and body.<side>-pinned; persist open + pinned"
  sidebar-pin:
    trigger: "click on [.pin-toggle][data-sidebar='left'|'right']"
    effect: "toggle .pinned and .open in lockstep; toggle body.<side>-pinned (pushes main content); swap pin ↔ pin-off icon; persist"
    applies: "≥{breakpoints.sidebar-pin}"
  panel-next:
    trigger: "click on [.carousel-chevron][data-dir='next']"
    effect: "setPanel(side, current+1); wraps forward past last with directional slide-through"
  panel-prev:
    trigger: "click on [.carousel-chevron][data-dir='prev']"
    effect: "setPanel(side, current-1); wraps backward past first with directional slide-through"
  panel-jump:
    trigger: "click on [.carousel-dot][data-panel=<n>]"
    effect: "setPanel(side, n) without wrap animation (direct translate or fade)"
  carousel-mode:
    trigger: "click on [data-carousel='slide'|'fade']"
    effect: "suppress transitions, apply mode-specific transforms/classes, force reflow, restore transitions; persist"
  sidebar-width:
    trigger: "change or blur on .sidebar-width-spinner"
    effect: "clamp to 10–100 and snap to step 5; set --sidebar-width-<side> on :root; persist"
  tree-toggle:
    trigger: "click on [.tree-toggle]"
    effect: "toggle .collapsed on closest .tree-branch; swap folder ↔ folder-open icon; persist array of expanded branch indices"
  dropdown-toggle:
    trigger: "click on [.dropdown-toggle]"
    effect: "close any other open dropdowns (singleton); toggle .open on this dropdown"
  dropdown-dismiss-outside:
    trigger: "click outside any [.dropdown]"
    effect: "close all open dropdowns"
  dropdown-dismiss-escape:
    trigger: "keydown Escape"
    effect: "close all open dropdowns"
  toast-show:
    trigger: "Base.toast(message, type, ms)"
    effect: "remove any existing .toast, append new one with role='alert' and a close × button; ms > 0 auto-dismisses after {ms} (default 3000), ms <= 0 is sticky until the × is clicked (use for errors); fade 300ms"
  sidebar-group-collapse:
    trigger: "click on .sidebar-group-title"
    effect: "toggle .collapsed on closest .sidebar-group"
  scroll-detect:
    trigger: "window scroll (passive)"
    effect: "toggle body.scrolled when scrollY > 0"
  scroll-reveal:
    trigger: "IntersectionObserver (single observer; rootMargin 0px 0px -8% 0px)"
    effect: "elements already in view at load get .active with the transition suppressed for one forced reflow and then restored (snap visible, no entrance animation, hover still animates); the rest get .active one-way as they scroll into view, then are unobserved. No-IntersectionObserver fallback: all get .active immediately"
  system-theme-change:
    trigger: "matchMedia('(prefers-color-scheme: dark)') change"
    effect: "if no explicit user theme persisted, swap html.dark ↔ html.light to match OS"
  viewport-shrink-below-desktop:
    trigger: "matchMedia('(min-width: {breakpoints.sidebar-pin})') leave"
    effect: "close all open sidebars and unpin both; DO NOT clear persisted pinned state (restored on return to desktop)"
  viewport-grow-to-desktop:
    trigger: "matchMedia('(min-width: {breakpoints.sidebar-pin})') enter"
    effect: "re-run Base.restore() to reinstate pinned/open sidebars from persisted state"
accessibility:
  prefers-color-scheme:
    respected: true
    override: "html.dark or html.light (manual) wins; persisted theme wins over system"
  prefers-reduced-motion:
    respected: true
    effect: "all animation-duration and transition-duration reduced to 0.01ms"
  prefers-contrast:
    respected: "partial — high-contrast borders darkened in :root section"
  focus-visible: "relied on for keyboard outline; never suppressed by framework"
  aria:
    toast: "role='alert' for live-region announcement"
    theme-toggle: "aria-label updated to 'Light mode' or 'Dark mode' per current state"
keyboard:
  Escape: "close all open dropdowns"
  Tab: "native focus order; framework never intercepts"
---

## Overview

DCS is a client-only state machine. There is no server, no framework, no reactivity library — one ~420-line `Base` object in `docs/base.js` owns all interactive state, one ~80-line `Site` object in `docs/site.js` adds marketing-layer enhancements (particles, scroll reveal, footer year). Every interaction is mediated through a single delegated `click` listener on `document`, plus a small set of `keydown`, `scroll`, and `matchMedia` listeners. State lives in one `localStorage` key (`base-state`) as a JSON blob.

Three axioms govern behavior:

1. **One source of truth per concern.** Theme lives in `html.dark` / `html.light`. Scheme lives in `html.scheme-<name>`. Sidebar state lives in `.open` / `.pinned` classes on the `.sidebar` element. Persistence is a write-through cache of those observable values — never the reverse.
2. **Side autonomy.** Left and right sidebars are independent. Each has its own open, pinned, panel-index, and width state. Opening one never affects the other.
3. **No frame.** No animation framework, no state library, no virtual DOM. All transitions are CSS; all state mutations are direct class toggles and `style` property writes. This keeps behavior auditable in under 500 lines.

## Motion & Timing

Five named durations and two eases, declared as CSS custom properties in `base.css`:

| Token | Value | Typical use |
|:------|:------|:------------|
| `duration-fast` | 150ms | color, opacity on hover; dropdown fade |
| `duration-base` | 200ms | buttons, links, form focus |
| `duration-slow` | 300ms | theme crossfade on `body`; toast-in; sidebar slide; main content push on pin |
| `duration-medium` | 400ms | service-card hover (transform + shadow); reveal |
| `duration-long` | 500ms | scroll-reveal transform + opacity |

| Ease | Curve | Feel |
|:-----|:------|:-----|
| `ease-out` | `cubic-bezier(0.33, 1, 0.68, 1)` | default; decelerates into rest |
| `ease-spring` | `cubic-bezier(0.34, 1.56, 0.64, 1)` | toast entrance only; slight overshoot |

Non-token timings hard-coded in JS:

- **Carousel wrap cleanup:** 320ms `setTimeout` after initiating a wrap-around slide. See `docs/base.js` lines 173–180. The number is deliberately slightly longer than the `duration-slow` on the `.panel-track` transform so the cleanup fires *after* the slide completes.
- **Toast auto-dismiss:** 3000ms display + 300ms fade-out. Caller can override the display time.
- **Toast fade-out transition:** 300ms before removal from DOM.

**Reduced motion:** `@media (prefers-reduced-motion: reduce)` collapses all animation-duration and transition-duration to 0.01ms (not 0 — zero breaks `transitionend` listeners). Carousel wrap still fires its 320ms timer; the user sees an instant jump instead of a slide, but state remains consistent.

## State & Persistence

All persisted state lives in `localStorage['base-state']` as a JSON object. The complete shape is documented in the `persistence.fields` token block above. Writes are eager: every interaction that changes state persists immediately via `Base.state({…})`.

Non-persisted state (ephemeral, recomputed on init):

- Dropdown `.open` class.
- Scroll-reveal `.active` class (set once per element via IntersectionObserver; one-way).
- Topnav `.scrolled` class on body.
- Lucide icon DOM nodes (re-rendered from `data-lucide` attributes).
- Hover, focus, pressed visual states (pure CSS).

**Persistence is not authoritative.** The DOM classes are the truth; `localStorage` is a restore cache. On init, `Base.restore()` reads the cache, applies classes, and the DOM takes over. This means external code that mutates classes directly still works; the next persisted write will reflect the new DOM state.

**Scheme "default" is special.** When `scheme` is `'default'`, no `scheme-*` class is on `<html>` and the Ocean palette (declared on `:root` in `site.css`) applies. Selecting another scheme adds exactly one `scheme-<name>` class. This is why scheme changes are safe in any order.

## Initialization & FOUC Prevention

Three phases, in strict order:

### Phase 1 — inline pre-script (in `<head>`)

A 5-line IIFE runs before any stylesheet loads, reads `localStorage['base-state']`, and sets `html.className = 'preload ' + theme + ' ' + scheme`. The `preload` class disables every transition via `html.preload, html.preload * { transition: none !important }`. This is the single most important line in `base.css` (line 82) — it prevents theme-swap flash during load.

Complete pre-script shape (see `CLAUDE.md` § FOUC Prevention for the literal string):

```js
(function(){
  var s = JSON.parse(localStorage.getItem('base-state')||'{}');
  var t = s.theme, c = s.scheme, h = document.documentElement;
  h.className = 'preload '
    + (t || (matchMedia('(prefers-color-scheme:dark)').matches ? 'dark' : 'light'))
    + (c && c !== 'default' ? ' scheme-' + c : '');
})();
```

### Phase 2 — `Base.init()`

Runs on `DOMContentLoaded` (or immediately if `readyState !== 'loading'`). Order:

1. `updateIcon()` — render the current theme's sun/moon into the toggle.
2. `restore()` — apply persisted sidebar widths, carousel mode, pinned/open state, panel positions, tree expansion.
3. Attach delegated `click` listener on `document`.
4. Attach `keydown` listener for Escape.
5. Attach `matchMedia` listeners for `prefers-color-scheme` and `(min-width: 1280px)`.
6. Attach per-input `change`/`blur` on sidebar-width spinners.
7. Attach passive `scroll` listener for topnav.
8. Render Lucide icons.
9. On the next animation frame, remove the `preload` class to enable transitions.

### Phase 3 — `Site.init()` (optional, marketing layer)

Runs independently on `DOMContentLoaded`. Injects 20 particle elements with randomized left/delay/duration, sets up an IntersectionObserver for reveal elements (snapping above-the-fold ones visible instantly), sets current year in `#year`. Has no dependency on `Base`.

The `requestAnimationFrame` in step 9 is load-bearing: it guarantees transitions are enabled *after* the final layout from `restore()` commits, so nothing animates during restore.

## Components

### Theme

Tri-state: explicit `light`, explicit `dark`, or undefined (follows system). System changes only propagate when undefined — persisted themes override OS preference deliberately, because a user who clicked the toggle expressed intent. Theme buttons (`[data-theme]`) set state explicitly; the legacy `.theme-toggle` icon button flips the current value. Body transitions background-color and color at `duration-slow` so the flip is perceptible but not distracting.

### Scheme

Six options: `default` (Ocean), `crimson`, `stone`, `forest`, `sunset`, `mono`. Changing scheme is a single class swap; there is no transition on hue because OKLCH shifts of 0.12 chroma × 100° hue look glitchy when animated across all elements simultaneously. Instant is correct here.

### Sidebar

Three states per side: closed, open (off-canvas overlay), pinned (push content, ≥1280px only). State transitions:

- **closed → open:** click `.menu-toggle[data-sidebar]`. Slides in at `duration-slow`.
- **open → pinned:** click `.pin-toggle[data-sidebar]` (only visible at ≥1280px). Adds `body.<side>-pinned`, which applies `margin-inline-<side>: <sidebar-width>` to `main`, pushing content.
- **pinned → closed:** click `.menu-toggle` closes *and* unpins. This is deliberate — `toggleSidebar` on an open sidebar removes both `.open` and `.pinned`, so menu click always means "go away" regardless of prior state. Without this, `restore()` would re-open the sidebar on next load.
- **viewport shrink below 1280px:** all sidebars force-close but persisted `pinned` state is preserved, so returning to desktop reinstates.

Sidebar width is user-adjustable per side, 10–100%, stepped by 10. At 100% a sidebar covers the entire viewport — an emergent full-page panel use case that the carousel pattern makes useful (stack multiple 100% panels for a deck-of-cards navigation).

### Panel Carousel

Each sidebar hosts a `.panel-track` containing N `.panel` children. Two transition modes, persisted globally (not per side):

- **Slide mode:** `.panel-track` translates horizontally by `-(index * 100%)`. Panels don't get `.active` class.
- **Fade mode:** track transform is cleared, sidebar gets `.fade-mode` class which switches panels to a CSS grid stack; only the `.active` panel is opaque.

Mode switching is a four-step dance (see `setCarouselMode`, lines 95–123):

1. Null out `transition` on track and panels.
2. Apply mode-specific transform/class.
3. Force reflow (read `track.offsetHeight`).
4. Restore default transitions.

This avoids the flash of a half-slid panel cross-fading while a half-faded panel slides.

**Directional wrap** (slide mode only): When navigating past the last panel, the destination panel is temporarily translated to `+(count × 100%)` (off the right edge) before the track slides to `-(count × 100%)`. After the slide completes (320ms), transitions are suppressed, the destination panel's transform is cleared, and the track snaps to the canonical position for the new index. Backward wrap mirrors this. A previously-queued cleanup is cancelled if another navigation happens mid-flight (see lines 150–154). This preserves the illusion of a continuous carousel without actually duplicating DOM.

### Tree Widget

Hierarchical list component. Each `.tree-branch` has a `.tree-toggle` that flips `.collapsed`, swaps the folder ↔ folder-open Lucide icon, and persists the current expanded-indices array. Indices are positional within a flat `querySelectorAll('.tree-branch')` — this means adding a branch higher in the tree shifts every persisted index below it. Trees are expected to be stable.

### Dropdown

Singleton: opening one dropdown closes all others. Dismissed by click outside any `.dropdown` or Escape key. Stops propagation on toggle click to prevent the outside-click handler from immediately re-closing it. CSS transitions opacity, transform, and visibility together at `duration-fast`.

### Toast

Singleton: calling `Base.toast()` removes any existing `.toast` before appending the new one. `role="alert"` triggers assistive-tech announcement. Default display 3000ms, fade 300ms. Toasts are ephemeral and not persisted.

### Sidebar Group

Collapsible grouping inside a sidebar panel. Click on `.sidebar-group-title` toggles `.collapsed` on the group. Not persisted (assumed to be navigational, always re-openable).

### Topnav

Fixed-position glassmorphic bar. Gains `body.scrolled` class the moment `scrollY > 0`, which triggers the `--nav-scrolled-bg` background variable. Passive listener — never blocks scroll.

### Reveal (site.js)

Elements with class `.reveal` start at `opacity: 0` and `translateY(100px) scale(0.95)`. At init, any already in view (`getBoundingClientRect().top < innerHeight`) are snapped to rest with no slide-up: the element's inline `transition` is set to `none`, `.active` is added, a reflow is forced (`void el.offsetWidth`), and the inline transition is then cleared so the stylesheet's own transition takes over again. The restore is load-bearing — the earlier `.reveal-instant` class set `transition: none !important` permanently, which also killed the hover lift on every card above the fold. The rest are watched by a single `IntersectionObserver` (`rootMargin: 0px 0px -8% 0px`) and get `.active` one-way the moment they scroll into view, triggering the 500ms transition; each is then unobserved. One-way — never removed even if scrolled off-screen. If `IntersectionObserver` is unavailable, all elements get `.active` immediately.

### Particles (site.js)

20 `div.particle` elements injected at init, each with `left: random%`, `animation-delay: random 0–15s`, `animation-duration: random 10–20s`. Pure CSS animation `float-up` drives the motion. Opacity follows `--particle-opacity` which varies by theme.

## Responsive Behavior

Four breakpoints, all `min-width`:

- **600px:** service-card padding grows; grids switch from 1-col to 2-col; card border-radius becomes visible.
- **900px:** 3-col grids activate for pricing and service variants.
- **1200px:** 4-col service grids; banner sections grow vertically.
- **1280px:** sidebar pin becomes possible. Below this, pin buttons are hidden and any pinned sidebar is force-closed. This is the *only* breakpoint handled in JS — all others are pure CSS.

The 1280px threshold is handled by a `matchMedia` change listener rather than a resize listener so it only fires at the crossing. On shrink, open sidebars are closed but their `pinned` persistence is preserved for restore on re-enlarge.

## Keyboard & Accessibility

- **Escape:** closes open dropdowns. Sidebars are deliberately *not* Escape-dismissible because pinned sidebars on desktop would lose state on every stray Escape press. Sidebars close via their own hamburger button only.
- **Tab:** native order. Framework never calls `preventDefault` on keyboard events except Enter on dropdown toggles.
- **Focus visibility:** relies on browser defaults and `:focus-visible`. Never suppressed.
- **ARIA:** toast has `role="alert"` for live-region announcement. Theme toggle updates `aria-label` to reflect the action it *will* take (`"Dark mode"` when currently light). Tree toggles, pin buttons, and sidebar menu buttons do not currently expose `aria-expanded` — this is a known gap, not by design.
- **Reduced motion:** all transitions + animations collapse to 0.01ms. Scroll-reveal still fires; users with reduced motion see the content appear instantly on scroll rather than slide/fade in.
- **Contrast:** `@media (prefers-contrast: more)` deepens border color. Not exhaustive; authors using high-contrast mode should audit per project.

## Do's and Don'ts

**Do**

- Treat `localStorage['base-state']` as a cache, not a source of truth. Mutating DOM classes directly is supported; the next `Base.state()` write will capture the new state.
- Use the delegated click listener for new interactions — add your `data-*` attribute, match it inside `Base.init()`'s click handler, and you get state persistence for free.
- Use `Base.toast()` for transient feedback. It's the only sanctioned ephemeral UI surface.
- Respect the side autonomy invariant. If you need coupled left/right behavior, compose it from two independent calls, don't invent cross-side state.
- Keep new persisted fields inside `base-state`. The single-key approach means all state restores atomically; splitting keys introduces restore-order bugs.
- Call `lucide.createIcons()` after any DOM insertion containing `[data-lucide]`. Icons don't self-render.

**Don't**

- Don't add Escape-to-close for sidebars. Mark has deliberately chosen hamburger-only dismissal; pinned-sidebar users would lose layout state to stray keypresses.
- Don't animate scheme or theme hue transitions. Instant is correct — attempting to crossfade 30+ OKLCH variables across every element looks broken in every browser that has tried.
- Don't clear persisted `pinned` state on viewport shrink. The user is still "a pinned-sidebar user"; they just happen to be on a phone right now.
- Don't add a resize listener where a `matchMedia` change listener will do. Resize fires at 60fps during drag; matchMedia fires once per crossing.
- Don't use `transitionend` to sequence things that need to happen after an animation. `prefers-reduced-motion` collapses durations to 0.01ms, which makes `transitionend` race with layout. Use `setTimeout` with a duration slightly longer than the animation (like the 320ms carousel cleanup), or force-reflow and read a layout property to guarantee commit.
- Don't write to `localStorage` on scroll or mouse-move. Every current write is tied to an intentional click or change event; preserve that.
- Don't forget the `preload` class. Without it, every page load flashes from the wrong theme on slow connections.
