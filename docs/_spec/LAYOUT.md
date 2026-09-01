---
version: alpha
name: DCS Layout
description: Spatial specification for the DCS (Dual Carousel Sidebars) reference implementation — HTML skeleton, positioning model, z-index scale, sidebar anatomy, section types, grid system, and pin-push geometry. Complements DESIGN.md (visual tokens) and BEHAVIOR.md (interaction) with the "how things are arranged" layer. Authoritative source is docs/index.html + docs/base.css.
layers:
  hero-overlay: 0
  particles: 0
  hero-content: 1
  sidebar-inner-border: 1
  dropdown: 100
  sidebar: 300
  menu-toggle: 301
  toast: 800
containers:
  lg: 960px
  xl: 1280px
  topnav-height: 4rem
  sidebar-width-default: 15%
  sidebar-width-min: 10%
  sidebar-width-max: 100%
  sidebar-width-floor: 200px   # clamp(200px, <pct>, 100%) — a rail is never narrower
  sidebar-width-step: 5        # spinners; drag is 1%
  menu-toggle-size: 48px
  menu-toggle-offset-top: 0.5rem
  menu-toggle-offset-side: 0.75rem
  menu-toggle-reserve: 3.75rem
breakpoints:
  tablet: 600px
  desktop-narrow: 900px
  sidebar-pin: 960px
  desktop: 1200px
positioning:
  body: "normal flow; background: fixed hero image (center/cover)"
  particles: "position: fixed; inset: 0; pointer-events: none"
  menu-toggle: "position: fixed; top: 0.5rem; left|right: 0.75rem; size 48×48"
  topnav: "position: relative (in flow); height 4rem — the centre column of the top row, scrolls away with content"
  sidebar: "position: fixed; top: 0; height: 100vh — its carousel header is the left/right column of the top row"
  sidebar: "position: fixed; top: 0; height: 100vh"
  sidebar-left-closed: "transform: translateX(-100%)"
  sidebar-right-closed: "transform: translateX(100%)"
  sidebar-open: "transform: translateX(0)"
  main: "normal flow; margin-inline reacts to pinned state"
  footer: "inside main; normal flow"
  toast: "position: fixed; top-inline-end; auto-dismiss"
  dropdown-menu: "position: absolute to its .dropdown ancestor"
skeleton:
  order:
    - "script (FOUC preload)"
    - "div.particles#particles"
    - "button.menu-toggle[data-sidebar='left']"
    - "button.menu-toggle[data-sidebar='right']"
    - "nav.topnav"
    - "aside.sidebar.sidebar-left"
    - "aside.sidebar.sidebar-right"
    - "main#content"
    - "script (base.js, site.js, md.js)"
sidebar-anatomy:
  root: "aside.sidebar (flex column, container-type: inline-size)"
  header:
    element: ".carousel-header"
    height: "4rem (matches topnav-height)"
    children:
      - ".carousel-nav (outer edge, right after the hamburger reserve)"
      - ".pin-toggle (INNER edge — left sidebar: last child + margin-inline-start:auto; right sidebar: first child + margin-inline-end:auto)"
    padding-reserve: "3.75rem on the side facing the menu-toggle, to clear the hamburger"
  carousel-nav:
    children:
      - ".carousel-chevron[data-dir='prev']"
      - ".carousel-dots (N .carousel-dot buttons)"
      - ".carousel-chevron[data-dir='next']"
  body:
    element: ".panel-viewport"
    flex: 1
    overflow: hidden
    child: ".panel-track"
  track:
    element: ".panel-track"
    display: flex
    transition: "transform 0.3s ease-in-out (slide mode); display: grid (fade mode)"
    children: "N .panel (width: 100%, flex-shrink: 0)"
  panel:
    structure:
      - ".panel-title (flex-shrink: 0, tinted header)"
      - ".panel-content (flex: 1, overflow-y: auto)"
sections:
  hero:
    class: "section.hero.hero-bg"
    height: "min-height: 100vh"
    overlay: "::before with rgb(0 0 0 / 0.6) dark, 0.3 light"
    content-layer: "z-index 1"
  content-section:
    class: "section.content-section"
    background: "var(--bg-secondary) solid"
    padding: "var(--space-16) var(--space-4)"
  bg-image-section:
    class: "section.bg-image-section"
    background: "transparent — body's fixed hero image bleeds through"
    overlay: "::before with rgb(0 0 0 / 0.5)"
    min-height: "30vh; 40vh at ≥1200px"
  footer:
    class: "footer.footer-main (inside main)"
    background: "var(--bg-secondary) solid"
    padding: "var(--space-12) var(--space-4) var(--space-8)"
grids:
  services-grid:
    base: "1 column"
    tablet: "2 columns"
    desktop: "4 columns"
  services-grid-2col:
    base: "1 column"
    tablet: "2 columns (stays 2)"
  services-grid-3col:
    base: "1 column"
    tablet: "2 columns"
    desktop-narrow: "3 columns"
  pricing-tables-grid:
    base: "1 column"
    tablet: "2 columns"
  pricing-tables-3col:
    desktop-narrow: "3 columns"
  footer-grid:
    base: "1 column"
    tablet: "2 columns"
    desktop: "4 columns"
pin-geometry:
  applies-at: "≥960px"
  pinned-sidebar: "transform: translateX(0); still position: fixed (does NOT push via transform)"
  main-left-pinned: "margin-inline-start: var(--sw-l)"
  main-right-pinned: "margin-inline-end: var(--sw-r)"
  main-both-pinned: "margin-inline: var(--sw-l) var(--sw-r)"
  topnav-left-pinned: "margin-inline-start: var(--sw-l)"
  topnav-right-pinned: "margin-inline-end: var(--sw-r)"
  pin-toggle-visibility: "display: none below 960px; display: block at ≥960px"
  first-visit: "no saved key → pinned on desktop (≥960px), closed below; saved state always wins"
---

## Overview

DCS's defining structural feature — the one it is named for — is two independent off-canvas sidebars, each containing a horizontal carousel of stacked content panels. Everything else in the layout is scaffolding around that claim.

Three structural axioms:

1. **Sidebars are siblings of main, not children.** The DOM order is particles → menu-toggles → topnav → left sidebar → right sidebar → main. This matters because it keeps the hamburger buttons stable regardless of pinning: they live at z-index 301, above sidebars, above main, so they remain clickable whether a sidebar is closed, open-as-overlay, or pinned-pushing-content.
2. **Topnav is in flow, not fixed.** Every other framework puts the top bar at `position: fixed`. DCS puts it at `position: relative` so that pinned sidebars can push it horizontally with `margin-inline`. Pinning is real layout, not `transform` fakery — content widths shrink to fit.
3. **The body's background image is the only decorative layer behind content.** `body { background: var(--hero-bg) center/cover fixed }` creates a single fixed backdrop. Opaque sections (`.content-section`, `.footer-main`) cover it; transparent sections (`.hero-bg`, `.bg-image-section`) with overlay `::before` pseudo-elements let it show through. There is no parallax — "parallax feel" comes free from `background-attachment: fixed` while content scrolls past.

## HTML Skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- fonts, base.css, site.css, lucide -->
  <script>
    (function(){var s=JSON.parse(localStorage.getItem('base-state')||'{}'),
    t=s.theme,c=s.scheme,h=document.documentElement;
    h.className='preload '+(t||(matchMedia('(prefers-color-scheme:dark)').matches?'dark':'light'))
    +(c&&c!=='default'?' scheme-'+c:'');})()
  </script>
</head>
<body>
  <div class="particles" id="particles"></div>

  <button class="menu-toggle" data-sidebar="left"><i data-lucide="menu"></i></button>
  <button class="menu-toggle" data-sidebar="right"><i data-lucide="menu"></i></button>

  <nav class="topnav">
    <h1><a class="brand" href="./">Your <span>Brand</span></a></h1>
  </nav>

  <aside class="sidebar sidebar-left">
    <div class="carousel-header">…</div>
    <div class="panel-viewport"><div class="panel-track">…panels…</div></div>
  </aside>

  <aside class="sidebar sidebar-right">
    <div class="carousel-header">…</div>
    <div class="panel-viewport"><div class="panel-track">…panels…</div></div>
  </aside>

  <main id="content">
    <section class="hero hero-bg">…</section>
    <section class="content-section">…</section>
    <section class="bg-image-section">…</section>
    <!-- … alternating content / bg-image sections … -->
    <footer class="footer-main">…</footer>
  </main>

  <script src="base.js"></script>
  <script src="site.js"></script>
</body>
</html>
```

**Order is load-bearing** in three places:

- **Pre-script in `<head>`** must run before stylesheets apply, to set the `preload` class and theme/scheme on `<html>`. Without this, page load flashes the wrong theme for one repaint.
- **`.particles` first in `<body>`** so it sits at the bottom of the stacking context. Its z-index is 0 and it's behind everything else in document order.
- **`.menu-toggle` buttons before `.topnav`** so they paint on top of the topnav background without needing a higher z-index (both live at z-index 301 on the menu buttons' side; order still matters for same-z stacking).

## Stacking & Z-Index

Five named layers, deliberately spaced with gaps for project-specific inserts:

| Layer | z-index | What lives here |
|:------|:--------|:----------------|
| particles / hero overlay | 0 | decorative background layer |
| in-section content, sidebar inner border | 1 | anything that needs to sit above an overlay |
| dropdown | 100 | dropdown menus |
| sidebar | 300 | both off-canvas sidebars |
| menu-toggle | 301 | hamburger buttons (stay clickable over sidebar) |
| toast | 800 | transient notifications |

Reserved gaps: 2–99 (section decorations), 101–299 (in-sidebar UI like sub-dropdowns), 302–799 (modals, popovers if added), 801+ (overlays on top of toast — avoid).

**Stacking gotcha:** `.menu-toggle` at z-index 301 is one above `.sidebar` at 300 so the hamburger remains clickable even when a sidebar is pinned and the toggle sits *inside* the sidebar's horizontal span. This is why the toggle buttons are `position: fixed` at page-level coordinates rather than children of either sidebar.

## Positioning Model

Everything fixed:

- **`.particles`** — `position: fixed; inset: 0; pointer-events: none`. Full viewport; never blocks clicks.
- **`.menu-toggle[data-sidebar]`** — `position: fixed; top: 0.5rem; left|right: 0.75rem; 48×48`. Deliberately 48px tap target for mobile.
- **`.sidebar`** — `position: fixed; top: 0; height: 100vh`. Left sidebar `left: 0` + `translateX(-100%)` when closed; right sidebar `right: 0` + `translateX(100%)` when closed. Opening removes the transform.
- **`.toast`** — `position: fixed; top: space-4; inset-inline-end: space-4`.

Everything in flow:

- **`.topnav`** — `position: relative; height: 4rem`. Critical: NOT fixed. Sits at the top of `<body>` immediately after the menu-toggles and before the sidebars.
- **`main`** — normal flow. Contains all content sections and the footer. Its `margin-inline` is the only thing that responds to pinned-sidebar state.

Absolute positioning appears only inside:
- `.dropdown-menu` relative to `.dropdown`
- `.hero-bg::before` overlay
- `.bg-image-section::before` overlay
- Sidebar internal border pseudo-elements

## Sidebar Anatomy

A sidebar is a two-row flex column:

```
.sidebar (flex column, fixed, 100vh, container-type: inline-size)
├── .carousel-header              ← row 1, height 4rem, matches topnav
│   ├── .carousel-nav             ← outer edge (after the hamburger reserve)
│   │   ├── .carousel-chevron[prev]
│   │   ├── .carousel-dots
│   │   │   └── .carousel-dot * N
│   │   └── .carousel-chevron[next]
│   └── .pin-toggle               ← INNER edge; left: last child, right: first child (mirrored)
└── .panel-viewport               ← row 2, flex: 1, overflow: hidden
    └── .panel-track              ← horizontal flex (slide) or stacked grid (fade)
        └── .panel * N            ← each 100% width
            ├── .panel-title      ← tinted strip under header
            └── .panel-content    ← scrollable body
```

**Mirror symmetry:** in the left sidebar, `.carousel-nav` is the *first* child of `.carousel-header` (flex-start, right after the `3.75rem` hamburger reserve) and `.pin-toggle` is *last* with `margin-inline-start: auto`; in the right sidebar it is mirrored — `.pin-toggle` first with `margin-inline-end: auto`, `.carousel-nav` last at flex-end before the reserve. So the carousel controls sit against the outer edge (near the viewport wall and the hamburger) and the pin sits against the **inner** edge, toward the main content — the boundary it reserves space from, where the resize handle also lives. Pinning only means anything while the sidebar is visible, so the control lives with the visible state, and the two control groups never compete for the narrow outer run (this mattered at 200px). DOM order matches visual order, so tab order is sane.

**The three-column top row is foundational.** Left carousel header · topnav · right carousel header, all `--topnav-height` tall. The two headers belong to the fixed sidebars and never move; the topnav is in normal flow and scrolls away with the content. Do not replace this with a fixed full-width topnav — that was tried on 2026-09-01 and reverted the same day.

**Container query:** `.sidebar { container-type: inline-size }` enables an internal breakpoint at `max-width: 230px` that collapses horizontal controls (toggle-group, width spinners, scheme dots) to a vertical stack. This is why a 10%-wide sidebar on a narrow viewport is still usable — the controls adapt to the sidebar's own width, not the viewport's.

**Panel height:** each panel is a flex column with a fixed-height `.panel-title` (flex-shrink: 0) and a scrollable `.panel-content` (flex: 1, overflow-y: auto). This means panel scrolling is local to the panel; the sidebar header and title stay pinned while panel content scrolls.

**Sidebar inner border** is a `::after` pseudo-element, 1px vertical line, positioned from `top: var(--topnav-height)` to `bottom: 0`. When `body.scrolled` is set (any scrollY > 0), it animates `top` to `0` so the border runs full-height. This visual continuity sells the "topnav is floating on top of the sidebars" effect even though the topnav is actually in flow *between* the two sidebars in the z-stack.

## Section Types

The content area is a vertical sequence of sections, alternating opaque and transparent:

### `.hero` + `.hero-bg`

The above-the-fold hero. `min-height: 100vh`, flex-centered. A dark overlay (`::before` with `rgb(0 0 0 / 0.6)` dark theme, `0.3` light) sits above the body's fixed hero image. Content in `.hero-content` is at z-index 1, above the overlay. Used once per page at the top.

### `.content-section`

Solid opaque section — the "reading surface." `background: var(--bg-secondary)`, padding `var(--space-16) var(--space-4)`. This is where cards, grids, and narrative prose live. Covers the body's fixed hero image completely.

### `.bg-image-section`

Transparent section that exposes the body's fixed hero image between content blocks — the "parallax" divider. `min-height: 30vh` (40vh at ≥1200px). Its `::before` adds a `rgb(0 0 0 / 0.5)` overlay for text contrast. Usually holds a single section title.

### `.footer-main`

Lives inside `main`, not as a sibling. `background: var(--bg-secondary)` solid, padding `var(--space-12) var(--space-4) var(--space-8)`, with a 4-column grid at desktop.

**Section rhythm:** the canonical page pattern is `.hero-bg` → (`.content-section` → `.bg-image-section`)* → `.footer-main`. Alternating opaque and transparent creates the rhythm of "read, breathe, read, breathe" as the user scrolls, with the fixed background image providing visual continuity across breath sections.

## Grid System

Four standard grids, all centered with `max-width: var(--container-xl)` (1280px) and `margin-inline: auto`:

| Grid | Mobile | Tablet (≥600) | Desktop-narrow (≥900) | Desktop (≥1200) |
|:-----|:-------|:--------------|:----------------------|:----------------|
| `.services-grid` | 1 | 2 | 2 | **4** |
| `.services-grid-2col` | 1 | 2 | 2 | 2 |
| `.services-grid-3col` | 1 | 2 | **3** | 3 |
| `.pricing-tables-grid` | 1 | 2 | 2 | 2 |
| `.pricing-tables-3col` | 1 | 2 | **3** | 3 |
| `.footer-grid` | 1 | 2 | 2 | **4** |

The grids deliberately skip some breakpoints. A 4-column services grid would feel cramped at 900px but expansive at 1200px+, so it jumps straight from 2 to 4. A 3-column grid needs the 900px step because 3 cards at 1200px are wide enough to breathe but 3 at 900px are ideal. Pick the variant that matches content density.

**Card padding responds to the grid:** `.service-card` grows from `var(--space-6)` to `var(--space-8)` at the 600px breakpoint, keeping visual weight proportional to cell size.

## Pin Geometry

Pinning is available at `≥960px` only. Below that, `.pin-toggle` is `display: none` and any pinned state in localStorage is ignored (see BEHAVIOR.md § Responsive). 960 is chosen so a tablet-class viewport (1024px) can pin both sides and still keep a 624px main column — the layout used for the SPE tutorial recordings at 4K.

**Crucial design choice:** a pinned sidebar does not use `transform` to push content — it remains `position: fixed` at `translateX(0)`. The push happens entirely via `margin-inline` on `main` and `.topnav`:

```css
body.left-pinned main       { margin-inline-start: var(--sw-l); }
body.right-pinned main      { margin-inline-end:   var(--sw-r); }
body.left-pinned.right-pinned main { margin-inline: var(--sw-l) var(--sw-r); }

body.left-pinned .topnav  { margin-inline-start: var(--sw-l); }
body.right-pinned .topnav { margin-inline-end:   var(--sw-r); }
```

Why separate margins on topnav and main rather than one margin on `<body>`? Because the topnav is `position: relative` in document flow *before* the sidebars in DOM order, but it visually sits next to pinned sidebars. Each needs its own horizontal push.

Width is driven by the clamped custom properties (200px floor, viewport-relative default):

```css
--sw-l: clamp(200px, var(--sidebar-width-left),  100%);
--sw-r: clamp(10%, var(--sidebar-width-right), 100%);
```

These are also what the pinned sidebar reads for its own `width` — so one user-facing spinner controls three things atomically: sidebar width, pushed topnav margin, and pushed main margin.

**Overlay vs push:** on mobile/tablet an open sidebar is an overlay (sits above content, does not push). On desktop an open-and-pinned sidebar pushes content. Open-but-not-pinned on desktop is still an overlay. The distinction is intentional — pinning is a committed gesture ("I want this visible while I work"); plain open is a transient gesture ("I'm looking something up, then closing").

## Background Image Strategy

A single declaration on `<body>`:

```css
body {
  background: var(--hero-bg) center/cover fixed;
  background-attachment: fixed;
  min-height: 100vh;
}
```

`background-attachment: fixed` is the whole trick. As the user scrolls, content moves but the background stays locked to the viewport. `.content-section` and `.footer-main` cover the background with solid fills; `.hero-bg` and `.bg-image-section` expose it through their transparent backgrounds with a dark overlay `::before`. The result: four visible appearances of the "same" image at consistent framing, no parallax plugin required.

Swap the image by overriding the `--hero-bg` custom property per-project (e.g. `--hero-bg: url('my-image.webp')`). Ensure it's at least 1920×1080 and under ~300KB for mobile; Server_Room_Dark.webp (the default) is the reference benchmark.

## Do's and Don'ts

**Do**

- Keep the skeleton order. Particles → toggles → topnav → sidebars → main is load-bearing for z-stacking and pinning geometry.
- Place the pre-script inline in `<head>` before any stylesheet. No deferred loading — it has to run before first paint.
- Use `.content-section` and `.bg-image-section` alternately for marketing pages. They are designed as a rhythmic pair.
- Reserve `3.75rem` of inline padding in `.carousel-header` on the side facing the menu-toggle. The hamburger lives there and will collide with carousel chevrons otherwise.
- Let the container query in `.sidebar` handle narrow widths. Do not add per-viewport media queries that duplicate what the internal `container-type: inline-size` already does.
- Centre content with `max-width: var(--container-xl)` + `margin-inline: auto`. Do not set explicit pixel widths on section containers.

**Don't**

- Don't make the topnav `position: fixed`. Pinning geometry depends on the topnav accepting `margin-inline`, which fixed positioning breaks.
- Don't push pinned sidebars with `transform` or `left/right` coordinates. They stay at `translateX(0) position: fixed` and push is applied to *main and topnav*, not to them.
- Don't place the menu-toggle buttons inside the topnav or sidebars. They live at page scope so they remain clickable under every pinning state and don't participate in the topnav's flex layout.
- Don't stack content at z-index ≥ 100 without reviewing this table. The named layers assume nothing else is competing in those ranges.
- Don't remove `background-attachment: fixed` from body. Every "parallax section" design below depends on it; switching to `scroll` kills the rhythm between opaque and transparent sections.
- Don't add more than ~4 panels per sidebar. The carousel works at any count, but chevron-and-dot navigation degrades past 5 — users lose track of where they are. For deeper hierarchies use the tree widget inside one panel rather than adding another panel.
- Don't use grids with 4+ columns below 1200px. The services-grid 2→4 jump skips 3 columns deliberately; do not reintroduce it.
- Don't nest a sidebar inside main or footer. DCS assumes sidebars are body-level siblings; nesting breaks z-index, fixed positioning, and pin geometry simultaneously.
