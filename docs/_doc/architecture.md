# Architecture

DCS uses a **two-layer system** that separates the generic framework from project-specific theming.

## Two-Layer Separation

| Layer | Files | Purpose |
|-------|-------|---------|
| **Base** | `base.css`, `base.js` | Generic framework — never modify per project |
| **Site** | `site.css`, `site.js` | Theme colors, marketing components — customize freely |

This separation means you can update the base layer independently without breaking your project's look and feel.

## CSS Cascade Layers

`base.css` declares six cascade layers in strict order:

```css
@layer reset, tokens, base, components, utilities, animations;
```

- **reset** — Box-sizing, reduced motion, contrast preferences
- **tokens** — Typography scale (6 sizes), spacing (0–8), radius, shadows, transitions, z-index
- **base** — Headings, links, code blocks, tables, blockquotes
- **components** — Container, cards, buttons, forms, dropdowns, toast, topnav, sidebars, prose, tree
- **utilities** — Display, flex, grid, spacing, text, glass morphism
- **animations** — Toast-in, fade-in, hover-lift

`site.css` adds its own layers that cascade *after* base:

```css
@layer site-tokens, site-components, site-utilities;
```

## HTML Structure

```
body
├── div.particles#particles
├── nav.topnav (fixed, glassmorphism)
├── aside.sidebar.sidebar-left (off-canvas)
├── aside.sidebar.sidebar-right (off-canvas)
├── main#content
│   ├── section.hero.hero-bg (100vh)
│   ├── section.content-section (solid bg, cards)
│   ├── section.bg-image-section (banner/divider)
│   └── footer.footer-main
├── script base.js
└── script site.js
```

## JavaScript Architecture

`base.js` exposes a single `Base` object with localStorage-backed state:

- **`state(updates)`** — Read/write persistent state as a single JSON object
- **`toggleTheme()`** — Switch between dark and light modes
- **`setScheme(name)`** — Apply a color scheme class to `<html>`
- **`toggleSidebar(side)`** — Open/close left or right sidebar
- **`pinSidebar(side)`** — Pin sidebar open on desktop (960px+)
- **`setPanel(side, index)`** — Navigate carousel panels
- **`toast(msg, type, ms)`** — Show a notification
- **`restore()`** — Rehydrate UI from localStorage on page load

All click handling uses a single delegated listener on `document`. No per-element event binding.

## FOUC Prevention

A small inline script in `<head>` reads localStorage *before* the page renders:

```html
<script>(function(){var s=JSON.parse(localStorage.getItem('base-state')||'{}'),
t=s.theme,c=s.scheme,h=document.documentElement;h.className='preload '+
(t||(matchMedia('(prefers-color-scheme:dark)').matches?'dark':'light'))+
(c&&c!=='default'?' scheme-'+c:'');})()</script>
```

The `preload` class disables all CSS transitions. `base.js` removes it after state is restored, preventing a flash of wrong theme or transition artifacts.
