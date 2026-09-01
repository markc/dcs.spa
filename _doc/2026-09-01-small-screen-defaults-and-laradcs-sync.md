# Small-screen defaults + laradcs sync

Brief for the next DCS update. Two workspaces, in order: **dcs.spa first**, then
**laradcs**. Written 2026-09-01 from a side-by-side of dcs.spa `docs/`, laradcs
`resources/{js,css}` and SPE root `base.css`/`base.js`/`site.css`.

## Why

There are three divergent DCS implementations and a fourth consumer (sshm-desktop,
Electron + React) about to start:

| Implementation | Stack | Generation | Notes |
|---|---|---|---|
| **dcs.spa** | vanilla CSS/JS | current | canonical reference — changes land here first |
| **laradcs** | Laravel + Inertia + React 19 + Tailwind 4 | one behind | canonical **React** port; upstream for React consumers |
| **SPE** (`markc/spe`, repo-root assets shared by all chapters + `docs/`) | PHP-rendered vanilla | two behind (pre-carousel) | tuned for a 1024-CSS-px viewport at 4K (DPR 3.75) for screen recording |

SPE's small-screen tuning is the version that has been proven on a real tablet-sized
viewport and stays legible in a 4K recording. **Decision (Mark, 2026-09-01): those
values become the defaults in dcs.spa and laradcs — not a preset, not a knob.**

Rule going forward: dcs.spa → laradcs → consumers vendor **byte-identical** and pin
the laradcs SHA. Consumer needs are exposed as props/config upstream, never patched
locally. Anything else is how we got three forks.

## Part 1 — dcs.spa: adopt SPE defaults

dcs.spa already has fluid `--text-*` `clamp()` tokens, the pre-paint `<head>` script
(`base-state`), and the narrow/normal/wide content mode. Three things change.

### 1.1 Pin breakpoint 1280 → 960

- `base.css` `@media (min-width: 1280px)` block (pin-toggle display, `.pinned`
  transforms, `body.*-pinned main` margins) → `960px`.
- `base.js` `restore()` `window.innerWidth >= 1280` → `960`; the `matchMedia`
  breakpoint listener likewise.
- `.sidebar-resizer` stays hidden below 768.
- Update every mention of "1280" in `CLAUDE.md`, `docs/_spec/BEHAVIOR.md`,
  `docs/_doc/architecture*`, `docs/index.html` prose.

### 1.2 Width model: px floor, viewport-relative default

Today: `--sidebar-width-left/right: 20%`, clamped 10–100 % via `--sw-l`/`--sw-r`,
user-adjustable (spinners step 5, drag 1 %). SPE: `clamp(200px, 15vw, 300px)`, fixed.

Recommended reconciliation (keeps per-side adjustability and the 100 % asymmetric
layouts, gains SPE's floor and default):

```css
--sidebar-width-left: 15%;
--sidebar-width-right: 15%;
--sw-l: clamp(200px, var(--sidebar-width-left), 100%);
--sw-r: clamp(200px, var(--sidebar-width-right), 100%);
```

- 1024 px viewport → 200 px each, 624 px main (identical to SPE).
- 1920 → 288 px. The 300 px cap only bites above ~2000 CSS px; drop it — users at
  that width can turn the spinner down, and a hard cap would make the existing
  "100 %" feature unreachable.
- Spinner min stays 10 % (the px floor wins below ~2000 px anyway); document that.
- **Open call for Mark:** keep the 300 px cap as well (`clamp(200px, …, 300px)`)
  and give up drag-to-100 %? Recommendation: no.

### 1.3 Geometry: sidebars below the topnav

Today: `.sidebar { top: 0; height: 100vh }`, the carousel header is
`--topnav-height` tall with 3.75 rem padding reserved for the hamburger, pinned
sidebars push the topnav (`body.left-pinned .topnav { margin-inline-start }`), and the
inner rail `::after` starts at `--topnav-height` and animates to 0 on `body.scrolled`.

SPE: `.sidebar { top: var(--topnav-height); height: calc(100vh - var(--topnav-height)) }`,
topnav spans the full width above both sidebars.

Adopting SPE's geometry deletes: the topnav push rules, the 3.75 rem header padding,
the `::after` rail top-offset + `body.scrolled` toggle (rail is simply `top: 0` within
the sidebar), and the scroll listener that drove it. Carousel header height becomes
its own token (it no longer has to match the topnav). Hamburgers stay in the topnav,
left and right ends, visible in both regimes.

Check the pinned state at 960–1279 px by hand — that range is newly pinnable.

### 1.4 Housekeeping

- `--topnav-height: 4rem` stays; consumers override it (sshm-desktop: 2.75 rem).
- Regenerate `docs/_spec/` (BEHAVIOR.md + any LAYOUT/DESIGN text that states
  breakpoints, widths, geometry). The spec must reproduce the new defaults.
- Journal entry in `_journal/2026-MM-DD.md`; note the dcs.spa SHA laradcs will sync to.

## Part 2 — laradcs: back-port to dcs.spa parity + consumer config

Files that carry DCS in laradcs (`resources/js/components/dcs/panel-carousel.tsx`,
`dcs/sidebar.tsx`, `contexts/theme-context.tsx`, `css/dcs/tokens.css`,
`dcs/panels/theme-panel.tsx`, `layouts/app/app-dual-sidebar-layout.tsx`).

### 2.1 Parity gaps (vs dcs.spa after Part 1)

| Area | laradcs today | Target |
|---|---|---|
| Width | one shared px width 200–500, slider | per-side, `15%` default, `clamp(200px, …, 100%)`, spinners step 5, drag 1 % |
| Breakpoint | 1280 (`matchMedia` + `xl:` classes) | 960 |
| Geometry | `fixed top-0 h-screen`, header aligned to topnav | below topnav (`top: var(--topnav-height)`) |
| Schemes | ocean, crimson, stone, forest, sunset | + **mono**; `default` naming as dcs.spa (or document the `ocean` alias explicitly) |
| Carousel wrap | `(i+1) % len` — snaps back across all panels | directional slide on wrap (dcs.spa `setPanel` offset trick) |
| Narrow sidebars | — | `container-type: inline-size` + dcs.spa's `@container (max-width: 230px)` reflow rules |
| First paint | `useState(loadState)` + effect → can flash | inline pre-paint script in the Blade root layout, same `base-state` shape |
| Type | fixed sizes | fluid `--text-*` clamp() tokens |
| Content width | — | narrow / normal / wide (`html.narrow|wide`) |
| Tree, toast | — | optional; port if a consumer needs them |

### 2.2 Consumer config (props on `ThemeProvider` / `Sidebar`)

Only what a consumer genuinely cannot get from defaults:

- `topnavHeight` (CSS length; sshm-desktop = `2.75rem`)
- `storageKey` (default `laradcs-state`; sshm-desktop = `sshm-state`)
- `defaults` for initial `theme` / `scheme` / per-side `{open, pinned, panel}`

Not config: breakpoint, width model, hamburgers, geometry. sshm-desktop's 900 px
window floor is simply in the overlay regime; a wider window is pinnable. Standard.

### 2.3 One-panel sidebar degrades to a plain sidebar

`PanelCarousel` with `panels.length === 1` must render no chevrons and no dots — just
the title and content. That is what lets SPE (single-panel sidebars by choice) rebase
onto canonical later without gaining UI it doesn't want. dcs.spa already carries the
"Legacy: sidebar > nav (for non-carousel sidebars)" block for the same reason.

### 2.4 Record the sync

`_doc/YYYY-MM-DD-dcs-sync.md` in laradcs: dcs.spa SHA it is level with, the file
list consumers vendor, and the config surface. Bump on every sync so "is laradcs
behind?" is a one-line check.

## Part 3 — consumers (after Parts 1–2)

- **sshm-desktop**: bump to React 19 / Tailwind 4 / electron-vite 5, vendor the
  laradcs DCS files at the recorded SHA into `src/renderer/src/dcs/`, pin the SHA in
  its CLAUDE.md, add a Mix `_bin/dcs-sync` that diffs the vendored tree against the
  pinned upstream (non-empty diff = local patch = fail; new SHA = re-vendor).
- **SPE**: rebase root `base.css`/`base.js` onto dcs.spa (single-panel sidebars,
  three schemes); should need zero layout overrides once Part 1 lands.

## Acceptance

1. dcs.spa at a 1024 × 576 CSS viewport: both sidebars pinnable, 200 px each, 624 px
   main, topnav full width above them, no horizontal scroll, Appearance panel usable.
2. dcs.spa at 900 px: overlay regime, hamburgers open/close, outside-click closes
   unpinned sidebars. At 1280+: unchanged behaviour apart from geometry.
3. SPE's current layout is reproducible from dcs.spa defaults with no CSS overrides.
4. laradcs: every row of §2.1 done; `panels.length === 1` renders no carousel nav;
   `topnavHeight` / `storageKey` props work; `_doc/…-dcs-sync.md` names the dcs.spa
   SHA.
5. A consumer can copy the laradcs DCS files unchanged and get a working shell by
   passing props only.
