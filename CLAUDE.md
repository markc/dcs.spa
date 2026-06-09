# CLAUDE.md

Project guidance for Claude Code.

## Overview

**DCS (Dual Carousel Sidebars)** is the canonical reference implementation of Mark's preferred web interface: dual off-canvas sidebars, each holding a carousel of panels, with OKLCH color schemes, dark/light modes, mobile-first layout, and marketing components. Zero dependencies (Lucide icons via CDN), no build step.

**This repo is the single source of truth.** Any project using DCS (plain PHP, static HTML, Laravel + React) references `~/.gh/dcs.spa` as the authority. When told "use the DCS interface", read these files and reproduce the pattern.

**Showcase:** [dcs.spa](https://dcs.spa) — a self-documenting SPA built with DCS (GitHub Pages).

## File Structure

Real files live in `docs/` (the Pages source). Root-level `base.css`, `base.js`, `site.css`, `site.js`, `md.js`, `ai.txt`, `Server_Room_Dark.webp` are **symlinks** into `docs/` for convenient reference from sibling projects.

```
docs/
├── index.html          # Self-documenting SPA showcase
├── base.css / base.js  # Generic framework — NEVER modify per-site
├── site.css / site.js  # Marketing theme (OKLCH colors, components) — customize
├── md.js               # Markdown renderer (for doc sites)
├── ai.txt              # AI content-context file (aitxt.ing spec)
├── _doc/               # In-page doc viewer: architecture, color-system, components, usage-patterns
├── _spec/              # Regeneration spec: DESIGN/LAYOUT/BEHAVIOR + per-scheme DESIGN-<scheme>-<mode>.md
├── *.svg / *.webp      # favicon, wordmark, screenshot, default background
├── CNAME / .nojekyll   # Pages config (custom domain; serve _doc/ unrewritten)
```

## Architecture

Two layers. Never modify the base layer per-site.

| Layer | Files | Role |
|-------|-------|------|
| **Base** | `base.css`, `base.js` | Generic, color-agnostic framework |
| **Site** | `site.css`, `site.js` | Theme colors + marketing components |

### base.css

CSS cascade layers: `reset, tokens, base, components, utilities, animations`. Defines **no colors** — all come from `site.css` custom properties. Mobile-first (single column, enhanced at 768px / 1280px). Cards are edge-to-edge on mobile, fully styled on desktop. Hamburger buttons are 48×48 and stay visible when sidebars are pinned.

Tokens include `--sidebar-width-left` / `--sidebar-width-right` (default 20%), clamped to 10–100% via `--sw-l` / `--sw-r`.

### base.js

Single `Base` object, state persisted in `base-state` localStorage. Public API: `state(updates)`, `toggleTheme()` / `setTheme(t)`, `setScheme(name)`, `toggleSidebar(side)` / `pinSidebar(side)`, `setPanel(side, i)`, `setCarouselMode(mode)`, `setSidebarWidth(side, pct[, snap])` / `applySidebarWidth(side, pct)`, `toast(msg, type, ms)`, `restore()`, `init()`. One delegated click listener; Lucide icons render after `restore()`.

### site.css

Layers: `site-tokens, site-components, site-utilities`. Extended spacing/typography, OKLCH color definitions, and marketing components (hero, sections, service cards, pricing, CTAs, particles, footer).

### Color System (OKLCH)

`oklch(L% C H)` — between schemes only the **hue** changes; L/C ratios stay constant. Each scheme has light + dark; `html.dark`/`html.light` override system preference. 6 schemes: **Ocean** (H=220, default), **Crimson** (25), **Stone** (60), **Forest** (150), **Sunset** (45), **Mono** (C=0, grayscale with colored status).

### md.js

Markdown-to-HTML renderer for doc sites. `md()` (string → HTML), `loadDoc()` (fetch + render). Add `data-md-auto` to a container for auto-loading. Supports headings, lists, code blocks, tables, links, images, emphasis.

## Key Components

- **Sidebar panel carousel** — each sidebar holds a horizontal carousel of panels (Navigation, Appearance, Tree/Docs). Slide (default) or fade transition; navigate via chevrons, dots, or `Base.setPanel(side, i)`.
- **Appearance panel** — built-in right-sidebar panel: theme toggle, carousel-mode toggle, sidebar-width number spinners, scheme dots.
- **Tree widget** — hierarchical nav (`.tree` → `.tree-branch`/`.tree-toggle`/`.tree-item`), depth via `--tree-depth`, collapse animates `grid-template-rows`. Used by the left-sidebar doc viewer rendering `docs/_doc/`.
- **Sidebar resize** — drag the inner edge of either sidebar to set its width (1% precision, 10–100%), or use the Appearance spinners (step 10). A JS-injected `.sidebar-resizer` handle wires pointer-drag → live width var; the final width persists to `base-state` and restores verbatim on load. Hidden below 768px.

### Icons

Lucide via unpkg CDN, pinned in `index.html` (currently `lucide@1.17.0`). Lucide 1.x **dropped brand icons** (e.g. GitHub) — those are inlined as `<svg class="lucide" fill="currentColor">` rather than `data-lucide`. When bumping Lucide, re-validate that every `data-lucide="…"` name still resolves.

## Usage Patterns

1. **Marketing site** — `base.css` + `site.css` + `base.js` + `site.js`.
2. **Documentation site** — `base.css` + `site.css` + `base.js` + `md.js`. Same stylesheets; unused marketing rules simply never render. Pick a scheme on init (`Base.setScheme('stone')`).
3. **Laravel + React (Inertia)** — import OKLCH tokens into Tailwind v4 `@theme`; use React context for theme state instead of `base.js`.

## Key Patterns

### FOUC Prevention
```html
<script>(function(){var s=JSON.parse(localStorage.getItem('base-state')||'{}'),
t=s.theme,c=s.scheme,h=document.documentElement;h.className='preload '+
(t||(matchMedia('(prefers-color-scheme:dark)').matches?'dark':'light'))+
(c&&c!=='default'?' scheme-'+c:'');})()</script>
```

### HTML Structure
```
body
├── div.particles#particles
├── nav.topnav (fixed, glass)
├── aside.sidebar.sidebar-left  (off-canvas)
├── aside.sidebar.sidebar-right (off-canvas)
├── main#content  (hero → content-section → bg-image-section → footer)
└── script base.js / site.js
```

### Relative Paths
Root pages use `href="base.css"`; subpages use `href="../base.css"`. Serveable from any depth.

### CSS Variable Contract
`site.css` MUST define (base.css defines none):
```
--bg-primary, --bg-secondary, --bg-tertiary
--fg-primary, --fg-secondary, --fg-muted
--accent, --accent-hover, --accent-fg, --accent-subtle
--border, --border-muted
--success, --danger, --warning
--glass, --glass-border
```

## Using DCS in Other Projects

1. Read this file for the canonical pattern.
2. Copy from `~/.gh/dcs.spa/docs/` (the real files, not root symlinks).
3. Customize `site.css` for colors/branding. **Never** touch `base.css` / `base.js`.
4. Deeper reference: `docs/_doc/` (architecture, color-system, components, usage-patterns); `docs/_spec/` for full regeneration specs.

For Laravel + React, adapt OKLCH tokens into Tailwind v4 `@theme` and use React context for theme state.

## Production Sites

- [dcs.spa](https://dcs.spa) — canonical showcase (Ocean) · [motd.com](https://motd.com) (Crimson) · [renta.net](https://renta.net) (Crimson) · [SPE](https://github.com/markc/spe) (Stone)
- [laradcs](https://github.com/markc/laradcs) — Laravel + Inertia + React starter kit

## License

MIT — Copyright © 2026 Mark Constable <mc@dcs.spa>
