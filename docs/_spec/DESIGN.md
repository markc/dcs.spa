---
version: alpha
name: DCS (Dual Carousel Sidebars)
description: Canonical reference implementation of Mark Constable's preferred web interface pattern — dual off-canvas sidebars with panel carousels, OKLCH-based theming, glass-morphism, and a mobile-first two-layer (base + site) CSS framework with zero dependencies.
colors:
  bg-primary: "#F5F8FA"
  bg-secondary: "#EEF3F6"
  bg-tertiary: "#DEE8EE"
  fg-primary: "#1F3040"
  fg-secondary: "#385166"
  fg-muted: "#5E7285"
  accent: "#4D7FA8"
  accent-hover: "#568EBD"
  accent-fg: "#FAFCFD"
  accent-subtle: "#DFEAF1"
  border: "#CDD6DC"
  border-muted: "#DDE3E7"
  success: "#2E7D4A"
  danger: "#C85038"
  warning: "#D4A23F"
  glass: "#F5F8FA"
  glass-border: "#CDD6DC"
typography:
  display:
    fontFamily: "Quicksand, sans-serif"
    fontSize: "clamp(2.5rem, 2rem + 5vw, 5rem)"
    fontWeight: 700
    lineHeight: 1.1
    letterSpacing: "-0.01em"
  h1:
    fontFamily: "Quicksand, sans-serif"
    fontSize: "clamp(1.75rem, 1rem + 3vw, 2.5rem)"
    fontWeight: 700
    lineHeight: 1.2
  h2:
    fontFamily: "Quicksand, sans-serif"
    fontSize: "clamp(1.5rem, 1rem + 2vw, 2rem)"
    fontWeight: 700
    lineHeight: 1.25
  h3:
    fontFamily: "Quicksand, sans-serif"
    fontSize: "1.125rem"
    fontWeight: 600
    lineHeight: 1.3
  body:
    fontFamily: "Inter, Quicksand, system-ui, sans-serif"
    fontSize: "1rem"
    fontWeight: 400
    lineHeight: 1.6
  small:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: "0.875rem"
    fontWeight: 400
    lineHeight: 1.5
  caption:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: "0.75rem"
    fontWeight: 600
    lineHeight: 1.4
    letterSpacing: "0.1em"
  code:
    fontFamily: "ui-monospace, SFMono-Regular, Menlo, Consolas, monospace"
    fontSize: "0.875rem"
    fontWeight: 400
rounded:
  none: 0px
  sm: 4px
  base: 8px
  md: 10px
  lg: 12px
  xl: 16px
  2xl: 24px
  full: 9999px
spacing:
  0: 0
  1: 4px
  2: 8px
  3: 12px
  4: 16px
  5: 20px
  6: 24px
  8: 32px
  10: 40px
  12: 48px
  16: 64px
  20: 80px
  24: 96px
components:
  button-primary:
    backgroundColor: "{colors.accent}"
    textColor: "{colors.accent-fg}"
    typography: "{typography.body}"
    rounded: "{rounded.lg}"
    padding: "{spacing.3} {spacing.6}"
    height: 44px
  button-primary-hover:
    backgroundColor: "{colors.accent-hover}"
    textColor: "{colors.accent-fg}"
    rounded: "{rounded.lg}"
  button-secondary:
    backgroundColor: "#1A1A1A1A"
    textColor: "{colors.fg-primary}"
    typography: "{typography.body}"
    rounded: "{rounded.lg}"
    padding: "{spacing.3} {spacing.6}"
    height: 44px
  card:
    backgroundColor: "{colors.glass}"
    textColor: "{colors.fg-primary}"
    rounded: "{rounded.lg}"
    padding: "{spacing.6}"
  service-card:
    backgroundColor: "{colors.glass}"
    textColor: "{colors.fg-primary}"
    rounded: "{rounded.lg}"
    padding: "{spacing.6}"
  topnav:
    backgroundColor: "{colors.glass}"
    textColor: "{colors.fg-primary}"
    typography: "{typography.body}"
    height: 64px
    padding: "{spacing.3} {spacing.4}"
  sidebar:
    backgroundColor: "{colors.glass}"
    textColor: "{colors.fg-secondary}"
    width: "20%"
    padding: "{spacing.4}"
  sidebar-nav-item:
    backgroundColor: "transparent"
    textColor: "{colors.fg-secondary}"
    typography: "{typography.body}"
    rounded: "{rounded.md}"
    padding: "{spacing.2} {spacing.3}"
  sidebar-nav-item-active:
    backgroundColor: "{colors.accent-subtle}"
    textColor: "{colors.accent}"
    rounded: "{rounded.md}"
  input:
    backgroundColor: "{colors.bg-secondary}"
    textColor: "{colors.fg-primary}"
    typography: "{typography.body}"
    rounded: "{rounded.md}"
    padding: "{spacing.2} {spacing.3}"
    height: 40px
  badge:
    backgroundColor: "{colors.accent-subtle}"
    textColor: "{colors.accent}"
    typography: "{typography.caption}"
    rounded: "{rounded.full}"
    padding: "{spacing.1} {spacing.3}"
  toast:
    backgroundColor: "{colors.glass}"
    textColor: "{colors.fg-primary}"
    rounded: "{rounded.md}"
    padding: "{spacing.3} {spacing.4}"
---

## Overview

DCS is a two-layer design system: a **base** layer (`base.css`, `base.js`) that defines a color-agnostic framework of tokens, components, and state management, and a **site** layer (`site.css`, `site.js`) that supplies OKLCH color schemes and marketing components. The tokens above express the default *Ocean* scheme in the light theme and are the contract other consumers (plain HTML, Laravel + React via Tailwind v4) re-implement in their own color space.

The system exists because Mark Constable kept rebuilding the same shell — dual off-canvas sidebars with panel carousels, a translucent fixed topnav, a 100vh hero, and a handful of marketing sections — across unrelated projects. DCS is the single source of truth; downstream sites should read this file and reproduce it rather than drift.

Two axioms govern every decision:

1. **Zero build step, zero dependencies.** Everything is hand-authored CSS + vanilla JS served statically. This constrains token count, keeps the framework reviewable in one sitting, and makes it trivial to embed in any stack.
2. **Mobile-first, then enhanced.** Base styles assume single-column, full-width, no radius. Radius, grid, and pinned sidebars are layered on at `600px`, `900px`, and `1200px` breakpoints.

### Variants

Eleven sibling files cover every `(scheme × mode)` combination. Each overrides `colors:` only and inherits typography, spacing, rounded, components, and all prose rationale from this file.

| Scheme | Hue | Light | Dark |
|:-------|:----|:------|:-----|
| Ocean (default) | 220 | *this file* | [`DESIGN-ocean-dark.md`](./DESIGN-ocean-dark.md) |
| Crimson | 25 | [`DESIGN-crimson-light.md`](./DESIGN-crimson-light.md) | [`DESIGN-crimson-dark.md`](./DESIGN-crimson-dark.md) |
| Stone | 60 | [`DESIGN-stone-light.md`](./DESIGN-stone-light.md) | [`DESIGN-stone-dark.md`](./DESIGN-stone-dark.md) |
| Forest | 150 | [`DESIGN-forest-light.md`](./DESIGN-forest-light.md) | [`DESIGN-forest-dark.md`](./DESIGN-forest-dark.md) |
| Sunset | 45 | [`DESIGN-sunset-light.md`](./DESIGN-sunset-light.md) | [`DESIGN-sunset-dark.md`](./DESIGN-sunset-dark.md) |
| Mono | (C=0) | [`DESIGN-mono-light.md`](./DESIGN-mono-light.md) | [`DESIGN-mono-dark.md`](./DESIGN-mono-dark.md) |

Authoritative OKLCH source for all variants: `docs/site.css`. Hex values in each file are sRGB approximations.

## Colors

The tokenised palette above is **sRGB hex** — the format DESIGN.md requires. The authoritative source-of-truth in the codebase is **OKLCH**, declared in `site.css`. Hex values are approximations of the Ocean light-theme OKLCH colors (hue 220, chroma 0.008–0.14). Consumers producing their own stylesheet should prefer the OKLCH values and treat hex as a fallback.

- `accent` is the single hue pivot. All six built-in schemes (**Ocean** H=220 default, **Crimson** H=25, **Stone** H=60, **Forest** H=150, **Sunset** H=45, **Mono** C=0) change only the hue; lightness and chroma ratios stay constant. This keeps contrast predictable across themes.
- `accent-subtle` exists so active nav, table row hover, and badges never rely on `accent` at full strength inside content areas — those contexts need legibility, not emphasis.
- `accent-glow` (alpha 0.3 of accent) is reserved for drop shadows around interactive cards and primary CTAs. It is the only place color bleeds outside its container.
- `glass` and `glass-border` are semi-transparent surfaces that pair with `backdrop-filter: blur(20px)` on the topnav and sidebars. They are the *only* surfaces meant to let the hero background image bleed through.
- `success`, `danger`, `warning` are semantic and rotate independently of the scheme hue. Mono scheme inherits them unchanged because pure grayscale cannot encode status.
- Dark theme is not a separate scheme — each scheme defines a light and dark variant, triggered by `html.dark` / `html.light` (manual toggle) or `@media (prefers-color-scheme: dark)` (system default).

## Typography

Two families, chosen for long-term readability rather than novelty:

- **Inter** (body) — optimised for UI at 14–16px, comfortable line metrics, no stylistic detours.
- **Quicksand** (headings, wordmark, brand) — rounded geometric sans, softer than Inter, intentionally distinct so `h1`–`h6` feel like titles without bold tricks.

`clamp()` drives every heading size so a single declaration covers mobile through desktop without media queries. The lower bound keeps headings readable on 360px phones; the upper bound stops 4K displays from inflating them past reading width.

- `caption` carries `letter-spacing: 0.1em` and `text-transform: uppercase` (applied via the `.section-tag` component). It is the only place tracking is loosened — reserved for small category labels above section titles.
- Body copy is `1rem / 1.6`. This is deliberately generous: DCS is often used for marketing pages where line-height under 1.55 looks dense at content widths of 600–800px.
- Monospace stays in code blocks only. No UI uses it — even tabular numerals.

## Layout

Layout is specified separately — see [`LAYOUT.md`](./LAYOUT.md) for HTML skeleton, z-index scale, positioning model, sidebar anatomy, section types, grid system, and pin-push geometry.

Values that inform token decisions elsewhere in this file:

- **Container widths:** `--container-lg: 960px` (base), `--container-xl: 1280px` (marketing).
- **Sidebar widths:** user-adjustable 10–100% per side (spinners step 5, drag 1%), default 15%, resolved as `clamp(200px, <pct>, 100%)` so a rail is never under 200px.
- **Breakpoints:** 600px (tablet), 900px (3-col grids), 960px (sidebar pin), 1200px (4-col grids).
- **Topnav height:** 4rem, `position: fixed`, full width above both sidebars. Sidebars start at `top: var(--topnav-height)`, so pinning pushes only `main` via `margin-inline`.
- **Sidebar header height:** 3rem (`--sidebar-header-height`) — independent of the topnav.

## Elevation & Depth

Three shadow tiers, light-theme:

- `shadow-sm` — `0 1px 3px rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)` — input focus, subtle chips.
- `shadow-md` — `0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)` — cards at rest.
- `shadow-lg` — `0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)` — dropdowns, hovered cards.

In dark theme each tier uses `0.4` alpha instead of `0.1` because dark surfaces need heavier shadow-to-background contrast to read as elevated. A fourth non-numeric effect, **colored glow** (`0 0 20px var(--accent-glow)`), attaches to `.service-card:hover` and `.cta-btn.primary` — it is the only shadow that carries brand hue, deliberately rare.

**Glassmorphism** is the project's signature depth device: `backdrop-filter: blur(20px)` over `--glass` on topnav and sidebars, `blur(10px)` on cards layered directly over the hero image. It is the entire reason the background image is declared `background-attachment: fixed` on `body` — every translucent surface should sample the same blurred plane.

## Shapes

- `rounded.sm` (4px) — form inputs, small chips.
- `rounded.base` (8px) — default for buttons on desktop.
- `rounded.md` (10px) — sidebar nav items, toast.
- `rounded.lg` (12px) — cards, CTA buttons. This is the dominant shape.
- `rounded.xl` (16px) / `2xl` (24px) — reserved for hero-adjacent containers on large viewports.
- `rounded.full` — pill badges, particle dots, hero badges.

**Mobile flattening:** cards render with `rounded.none` and no border below 600px. This is a load-bearing decision — stacked full-bleed cards on a phone should feel like content blocks, not floating objects. Radius is an affordance that only makes sense once a card has horizontal breathing room.

## Components

Every component above composes tokens; none hard-code values. Notes on the non-obvious ones:

- **`topnav`** — fixed, glassmorphic, height 64px, full width above both sidebars. No scroll-state class (the `--nav-scrolled-bg` tokens remain defined for sites that want one).
- **`sidebar`** — glassmorphic off-canvas panel hosting a horizontal carousel of sub-panels (Navigation, Appearance, Tree/Docs). Two transition modes: **slide** (translateX, default) and **fade** (opacity + grid stacking). The distinction matters because fade avoids horizontal overflow when panels contain wide content like documentation.
- **`sidebar-nav-item-active`** — uses `accent-subtle` background + `accent` text. Same pattern as `badge` intentionally: both are "marked, not emphasised" states.
- **`service-card`** — has a zero-opacity 3px top accent stripe that fades in on hover alongside a `accent-glow` box-shadow and a −4px translate. The lift is the only place the design breaks pure material-style flat layering.
- **`button-primary`** — always gets `0 4px 20px var(--accent-glow)` at rest; the shadow is a permanent part of its identity, not a hover affordance. Hover only deepens it.
- **`cta-btn.secondary`** — the one component that uses literal white alphas (`rgb(255 255 255 / 0.1)`) instead of tokens, because it is exclusively placed over the dark hero image. Do not reuse it on solid content surfaces.

Hover, active, and focus variants are realised in `base.css` with `:hover` / `:active` / `:focus-visible` selectors against the same tokens — no separate component entries needed except where the hover state is visually distinct enough to be designed separately (e.g. `button-primary-hover`).

## Do's and Don'ts

**Do**

- Read `~/.gh/dcs.spa/CLAUDE.md` before reproducing DCS in another project. It is the prose companion to this token file.
- Copy from `docs/` (the real files), not the root symlinks.
- Change `site.css` only. Keep `base.css` and `base.js` untouched — they are the framework.
- Prefer OKLCH when authoring new schemes. Change only the hue; keep the lightness/chroma ladder from Ocean.
- For documentation sites, use the same `base.css + site.css` pair as a marketing site. Omit the marketing HTML (hero / service cards / pricing / CTAs) and those rules never render — there is no runtime cost.
- Use `accent-subtle` for backgrounds of active/selected states. Use `accent` only for things that are themselves interactive emphasis (primary buttons, links, section tags).
- Render mobile first with no radius on cards; let desktop enhance.

**Don't**

- Don't introduce new color tokens outside the documented set. If a page needs a color `base.css` doesn't expose, that is a signal to extend the token layer, not to inline a value.
- Don't modify hex-to-OKLCH mappings without regenerating both halves of a scheme (light *and* dark). The schemes are paired; editing one side produces contrast drift.
- Don't add a build step. No Sass, no PostCSS, no bundler. The framework must stay readable and servable as plain files.
- Don't use Filament or Livewire in downstream projects — all DCS consumers are on Inertia v2 + React, or plain HTML.
- Don't use Docker. Containers are Incus or Proxmox only.
- Don't mix `docs/` (GitHub Pages source) with project `_doc/` or `_journal/` conventions — `docs/` is reserved for published output, underscore-prefixed directories are for working documentation.
- Don't rely on glass surfaces outside of the topnav, sidebars, and directly-over-hero cards. Content sections use solid `bg-secondary`; glassmorphism over solid backgrounds is just noise.
