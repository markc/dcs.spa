# Usage Patterns

DCS supports three integration modes depending on your project type — but the first two share the same two stylesheets. The difference is which HTML components you mount.

## 1. Marketing / Brochure Site

Full design system with themed marketing components:

```html
<link rel="stylesheet" href="base.css">
<link rel="stylesheet" href="site.css">
<script src="base.js"></script>
<script src="site.js"></script>
```

This gives you the hero section, service cards, pricing tables, CTAs, particle effects, and scroll-reveal animations.

## 2. Documentation Site

Same two stylesheets as a marketing site — swap the marketing `site.js` for `md.js`:

```html
<link rel="stylesheet" href="base.css">
<link rel="stylesheet" href="site.css">
<script src="base.js"></script>
<script src="md.js"></script>
```

If your HTML never mounts the hero / service-card / pricing / CTA components, those rules simply do not render — there is no "marketing penalty" at runtime. Pick a scheme via `Base.setScheme('stone')` (or similar) on init, or let users switch it from the Appearance panel.

`md.js` provides `md()` for converting markdown to HTML and `loadDoc()` for fetching and rendering `.md` files. Add `data-md-auto` to opt in to automatic document loading:

```html
<div data-md-auto></div>
```

## 3. Laravel + React (Inertia v2)

Import OKLCH color tokens into Tailwind v4's `@theme` block:

```css
@theme {
    --color-accent: oklch(55% 0.12 220);
    --color-bg-primary: oklch(98% 0.005 220);
    --color-fg-primary: oklch(20% 0.02 220);
}
```

Use React context for theme state instead of `base.js`. The color math stays identical — only the delivery mechanism changes.

## FOUC Prevention

Every DCS page needs this inline script in `<head>`, *before* any stylesheet:

```html
<script>(function(){var s=JSON.parse(localStorage.getItem('base-state')||'{}'),
t=s.theme,c=s.scheme,h=document.documentElement;h.className='preload '+
(t||(matchMedia('(prefers-color-scheme:dark)').matches?'dark':'light'))+
(c&&c!=='default'?' scheme-'+c:'');})()</script>
```

This prevents a flash of the wrong theme by reading localStorage synchronously before the browser paints.

## Relative Path Strategy

DCS uses relative paths so pages work at any directory depth:

- Root pages: `href="base.css"`
- Subpages: `href="../base.css"`
- No absolute paths or CDN URLs required

## Sidebar Configuration

### Sidebar Width

Left and right sidebars are sized independently via `--sidebar-width-left` and `--sidebar-width-right` (default: `30%` each, range 25–75%). The Appearance panel exposes one slider per side, and values persist in localStorage. A `clamp(260px, …, 75vw)` wrapper keeps narrow viewports usable. Pinning both sides at 50% produces an equal split-screen layout (main content collapses to zero).

### Pinning

On viewports >= 960px, sidebars can be **pinned** open. The main content area and the topnav adjust their margins to accommodate pinned sidebars. Pinned state persists across page loads; a first visit starts with both sidebars pinned on desktop and closed below 960px.

### Carousel Panels

Each sidebar contains a panel carousel. Two transition modes are available:

- **Slide** — panels translate horizontally (default)
- **Fade** — panels cross-fade using opacity + grid stacking

Navigate with chevron arrows, dots, or programmatically via `Base.setPanel(side, index)`.

## Production Sites

| Site | Scheme | Type |
|------|--------|------|
| [dcs.spa](https://dcs.spa) | Ocean | Self-documenting showcase |
| [motd.com](https://motd.com) | Crimson | AI via email |
| [renta.net](https://renta.net) | Crimson | Managed Linux hosting |
| [SPE](https://github.com/markc/spe) | Stone | PHP tutorial documentation |
