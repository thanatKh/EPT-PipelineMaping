# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, static web app that visualises In-Line Inspection (ILI) anomaly
data for a PTT OR 18-inch subsea pipeline (T61 to Sea Berth, 2.7 km) on an
interactive Leaflet.js map with a synchronised elevation profile. No build
step, no npm — pure HTML/CSS/JS plus one data file.

Leaflet and the UI font are still vendored locally, but the app is **no longer
fully offline-capable**: it now pulls Tailwind CSS from the Tailwind Play CDN
(`https://cdn.tailwindcss.com`) at runtime, which requires internet on load and
prints a "should not be used in production" console notice. The live satellite/
street map tiles also require internet. Important nuance: Tailwind is wired in
(CDN script + a `tailwind.config` defining the slate theme, `Google Sans`, and
the semantic `prio.*` anomaly colours), but the actual styling still lives in
the inline `<style>` block — a CSS-variable design system in a **neutral-slate,
flat-solid** style: opaque panels, crisp 1px borders, subtle dark drop shadows
for elevation, and deliberately **no glassmorphism blur or glow**. (The
`--glass`/`--glass2` variable names are now opaque solids — kept for
compatibility with the many rules that reference them; `--blur` is set to
`none`.) No element currently consumes Tailwind utility classes, so the app
still renders fully styled even if the CDN is blocked; the utilities are simply
available for future markup. To restore true offline
operation, remove the CDN `<script>` (the inline styles stand alone) or compile
Tailwind to a vendored CSS file via the Tailwind CLI.

## Design standard (do not regress)

The UI standard is a **dark, flat-solid, high-contrast (projector-safe)** theme.
This app is used both at the desk **and projected in meetings**, so contrast is a
hard requirement, not a preference — dark mid-tone accents wash out badly on a
projector in a lit room. When changing any colour, keep these rules:

- **Contrast floor:** body/label text and accent text must clear **WCAG AA on the
  slate panels** (≈`#0f172a`). Current ratios: primary text 16:1, secondary
  (`--text2` slate-300) 12:1, muted (`--text3` slate-400) 7:1, accent text
  (`--blue2` blue-300 `#93c5fd`) ~10:1. Do **not** drop accent text back toward
  blue-500 `#3b82f6` (only ~4.5:1) — `#3b82f6` (`--blue`) is for *fills, lines,
  the slider and the route*, never for text on dark.
- **No glow, no blur, no glassmorphism.** Surfaces are opaque (`--glass`/
  `--glass2`/`--card` solids); depth comes from crisp 1px borders + subtle dark
  drop shadows only. `--blur` stays `none`.
- **Selected-state convention (hybrid):**
  - *Multi-select* controls (repair-priority toggles, defect-type & map-layer
    pills, report filter/type chips) = **soft fill, borderless** — accent-tint
    background + `--blue2` text, no outline. A spring-animated `✓` checkmark
    (via `::before` pseudo-element) appears when selected to reinforce state.
  - *Single-select* controls (Reports modal tabs, base-map segmented control,
    damage-map window presets) = **solid bright-blue fill + dark text**
    (`var(--sel-bg)` / `var(--sel-txt)`, ~7:1) so the active item is
    unmistakable when projected. No checkmark on single-select.
  - Never indicate selection with an accent **border outline** alone.

## Files

- `index.html` — the entire app: markup, CSS, and all JS logic (map setup,
  filters, sidebar, elevation chart, measurement tool, detail panels, CSV
  export). Styling is the inline `<style>` block driven by `:root` CSS
  variables (slate chrome + semantic anomaly colours) — change the palette
  there in one place. The Tailwind Play CDN `<script>` and its `tailwind.config`
  sit in `<head>` (theme tokens only; not yet used by any element). Note ~32
  class names (`.dp-row`, `.dp-badge`, `.depth-bar-fill`, `.rprio-dot`,
  `.leg-row`, `.pipe-label`, …) and several inline `var(--…)` colours are
  emitted from the JS template strings, so those class/variable names must not
  be renamed or removed without updating the JS too.
  Key CSS variables added recently: `--sel-bg`/`--sel-txt` for the single-select
  active fill (used by `.lseg-btn.on`, `.rtab.active`, `.pm-w.active`); `--r`/
  `--r-sm` for border-radius tokens. The `.lpill-full` modifier class handles
  full-width action pills (Measure/Export buttons) without inline styles.
- `data.js` — all anomaly/route data, loaded globally via `<script src="data.js">`.
  Exports `ANOMALIES`, `DENTS`, `ROUTE`, `REFS`, `ROUTE_PROFILE`.
- `README.md` — full feature documentation, data field reference, and update
  instructions for future ILI runs. Read it before changing data handling —
  it documents every anomaly object field and the meaning of `repair`/`depth`/
  `erf` classifications.
- `logo.png` — branding image used in the top bar (2033x827px; the `<img>`
  tag's `width`/`height` attributes must keep this ~2.46:1 aspect ratio or
  the logo will render squashed).
- `vendor/` — self-hosted Leaflet 1.9.4 (`leaflet/leaflet.js`, `leaflet.css`)
  and a latin-subset Google Sans variable webfont (`fonts/google-sans.css`
  + 2 woff2 files, ~70 KB total). Only the JS/CSS/font files are vendored —
  Leaflet's default marker/shadow/layers-control images are deliberately
  *not* included because this app supplies its own custom icons everywhere
  (`mkLbl`, `valveIco`, `measureDotIcon`, `measureLabelIcon`) and never
  instantiates `L.Icon.Default` or `L.control.layers`, so those assets would
  never be requested. See README.md for the curl commands to refresh these.

## Running locally

Browsers block `file://` script imports, so `data.js` must be served over HTTP:

```bash
python3 -m http.server 8080
# then open http://localhost:8080/index.html
```

(or `npx serve .`, or VS Code Live Server). There is no build, lint, or test
tooling in this repo.

## Architecture (all in index.html)

The script is one long top-to-bottom sequence of globals and functions
operating on the Leaflet `map` instance and the data arrays from `data.js`:

- **Map & layers**: `map`, `tiles` (satellite/street toggle), `routeLine`/
  `routeGlow` (centreline), `valveLayers` (reference points), `pLayers`
  (one Leaflet layer group per repair-priority bucket, keyed by the
  `RENDER_ORDER` array so critical anomalies always render on top of the
  green mass), `dentLayer`.
- **Colour/label helpers**: `RCOL`, `repairColor`, `depthColor`,
  `repairLabel`, `altZone` — central place to change colour coding or
  thresholds for repair priority, depth %, and bathymetric zones.
- **Filter predicate**: `passesActiveFilters(a)` is the single source of
  truth for "is this anomaly currently visible" (location, repair priority,
  min-depth). It drives `rebuildMarkers()`, `drawElev()`'s anomaly loop and
  hit-testing, and `exportFilteredCSV()` — change filter logic here and all
  three stay in sync.
- **Marker rendering**: `rebuildMarkers()` is the core function that clears
  and repopulates all marker layers based on current filter state
  (`activeRepair`, `showExternal`, `showInternal`, `showDent`, `minDepth`).
  Call it whenever a filter changes — and also call `drawElev()` (guarded by
  `if(elevOpen)`) so the elevation chart stays in sync with the map.
  `dentShouldShow()`/`syncDentLayer()` handle the dent layer specifically.
  Marker radius is `2.5+depthScale(a.depth)*3.5` (~3.2-5.9px); `depthScale`
  is the shared sqrt curve also used to size chart anomaly dots, so size
  means the same thing in both views.
- **Tooltips & detail panel**: `makeTooltip(a, col)` builds the hover
  tooltip HTML; `showDetail(a, col)` renders the full data card (wall-loss
  bar, elevation zone badge) when a marker is clicked. The panel is
  `position:fixed;top:84px;right:24px` (below the 72px topbar). Calling
  `showDetail` also calls `markSelectedPoint(a,col)` which places a pulsing
  `.search-ring` locator marker on the map; `closeDetailPanel()` clears it
  via `clearSelectedPoint()`.
- **Sidebar**: collapsible via `#sidebar`/`#sidebar-toggle`. Contains all
  filter toggles, "Live Statistics — Current View" (reactive to filters —
  distinct from the fixed "Survey Totals" KPI row in the topbar, which is
  always the full 22-23 Nov 2025 report total), and the
  "Export Filtered (CSV)" button (`exportFilteredCSV()`, `csvCell()`).
- **Measurement tool**: `measuring`, `measurePts`, `onMeasureClick`,
  `clearMeasure`, `stopMeasuring`, `haversine`, `formatDist` — click-two-
  points distance measurement drawn directly on the map.
- **Elevation profile**: `drawElev()` renders a canvas-based bathymetric
  profile chart from `ROUTE_PROFILE` and `ZONES` (depth-band colour ranges).
  - It sizes the canvas bitmap to `canvas.clientWidth/clientHeight` (its own
    rendered box) — **not** the wrapper's `clientWidth/clientHeight`, which
    would include the wrapper's padding and stretch the bitmap, throwing off
    both the drawn projection and the mousemove/click hit-testing (which
    reads CSS-pixel `getBoundingClientRect()` coordinates and compares them
    against the same `toX`/`toY` values used for drawing).
  - `toX(dist)`/`xToDist(canvasX)` project chainage ↔ canvas pixel by running
    every `ROUTE_PROFILE` point through Leaflet's own
    `map.latLngToContainerPoint`, then interpolating by `dist` and rescaling
    `[0, mapW] → [PAD.left, PAD.left+cW]`. This is geometrically exact (a
    point at the map's horizontal-center fraction lands at the chart's
    horizontal-center fraction) regardless of pan/zoom — verified to
    sub-pixel agreement against actual marker positions. Note the chart's
    plot area is narrower than the map (it reserves `PAD.left=46px` for the
    altitude axis labels), so a marker and its chart dot will *not* sit on a
    literal vertical line through the page — that's by design, not a bug.
  - Anomaly dots are drawn in `RENDER_ORDER` order (mirrors `pLayers`'
    z-stacking) so the rare critical `<=1yr` reds are never painted over by
    the abundant `>10yr` green mass — canvas has no z-index, last-drawn wins.
  - `elevHighlight(dist, alt, col)` draws a synced dashed crosshair + dot
    between map and chart on marker click; `toggleElev()` shows/hides the
    panel.
- **Responsive layout**: `fixMapHeight()` plus media queries in the
  `<style>` block handle tablet/mobile breakpoints (topbar reflow, sidebar
  auto-collapse on mobile).

## Reports & Analytics modal (also in index.html)

A second, self-contained UI surface opened via the `#btn-reports` card in the
sidebar (its own card above Search, separate from "Map Layers"). It's a
full-viewport overlay (`#reports-overlay` / `#reports-modal`, `z-index:4000`)
with its own native-fullscreen support (`toggleFS`, `#reports-fs`,
`fullscreenchange`), independent from the map/elevation chart below it.

- **Data source**: `REPORT_FEATURES = ANOMALIES.concat(DENTS)` — a single
  combined array so the table, charts, and damage map all iterate one list
  (dents carry `repair='<=1yr'` and `type==='dent smooth'` as their
  discriminator).
- **4 tabs** (`.rtab` / `.rpane`, switched by a click handler around line
  2147 that toggles `.active` and calls `requestAnimationFrame(drawAllCharts)`
  the first time a chart tab is opened — `chartsBuilt` guards re-init):
  - `pane-table` — 📋 Repair Action List: sortable/filterable table
    (`RCOLS`, `rSorted()`, `rFiltered()`); includes a free-text search box
    (`#rtable-search`, `rMatchesSearch()`) that filters across ID, type,
    surface, orientation, and comment. Row click calls `showFeatureOnPipeMap(a)`,
    which switches to the Damage Map tab, centers its zoom window on that
    feature, and highlights it.
  - `pane-pipemap` — 🗺 Damage Map ("unrolled pipe development"): an
    axial-distance × clock-position view drawn at true scale. Two canvases,
    `#pm-overview` (full-line context strip) and `#pm-detail` (zoomed
    window) via `drawPipeOverview()`/`drawPipeDetail()`/`drawPipeMap()`.
    Toolbar has window-width presets (50/100/250 m / full line, `pmWinWidth`/
    `pmWinStart`/`pmClamp()`), a measurement tool (`pmMeasure`,
    `pmMeasurePts`, reports Δaxial/Δcircum/surface distance using
    `PIPE_CIRC = Math.PI*457.2`), and a Prev/Next stepper
    (`pmStep`/`updateStepLabel`) to walk through the currently filtered
    feature list. Dents have no clock position, so they render as full-height
    pink bands at their chainage and are exempt from the %WT depth filter
    (they use % OD instead). Clicking a drawn box highlights it
    (`pmBoxes` hit-test registry) and shows a tooltip.
  - `pane-distance` — 📈 "Along Distance" charts: Wall Loss
    (`drawDepthChart`/`chart-depth`), ERF (`drawErfChart`/`chart-erf`),
    Metal-Loss Density (`drawDensityChart`/`chart-density`), Orientation
    (`drawOrientChart`/`chart-orient`).
  - `pane-profiles` — 📊 "Distributions" charts: Depth Distribution
    (`drawDepthHistChart`/`chart-depthhist`, ext/int histogram),
    Circumferential Distribution / rose chart (`drawRoseChart`/`chart-rose`),
    Defect Type (`drawTypeChart`/`chart-type`, POF `dim` classes).
  - All canvases use `cvSetup()` for `devicePixelRatio`-aware bitmap sizing,
    `attachScatterHover()`/`placeTip()`/`chartHit{}` for shared
    hover-tooltip wiring, and `.rc-fs` per-chart fullscreen buttons.
- **One shared filter bar** (`#rchart-filters`, hidden on the table tab via
  `.hidden`) drives every chart and the damage map — NOT duplicated per tab,
  so filter state persists when switching between chart tabs:
  ```js
  let chExt=true, chInt=true, chMinDepth=0, chTypes=null;
  const featType = a => a.type==='dent smooth' ? 'Dent' : a.dim;
  const chPass = a => (a.location==='External'?chExt:a.location==='Internal'?chInt:true)
                   && (a.type==='dent smooth' || (a.depth||0)>=chMinDepth)
                   && (!featType(a) || !chTypes || chTypes.has(featType(a)));
  ```
  `chTypes` is a `Set` of POF `dim` short-codes plus `'Dent'`, built by
  `buildTypeChips()` from `ALL_DIMS`. Every chart-drawing function filters
  `REPORT_FEATURES` through `chPass` — if you add a new chart or change filter
  semantics, update `chPass` once and all consumers stay in sync (mirrors the
  `passesActiveFilters` pattern used by the main map).
- **Entry point**: `drawAllCharts()` (around line 2126) is the single
  redraw-everything function, called on filter change, tab switch, and
  window resize. It does **not** include a KPI strip — that was tried and
  removed; don't re-add a `#rchart-kpis`/`updateKpis()` pattern without
  reason.

## Updating data for a new ILI run

Per the README: only `data.js` needs to change — replace `ANOMALIES`,
`DENTS`, `ROUTE`, and `REFS` from the new Pipetally export, and regenerate
`ROUTE_PROFILE` (interpolated altitude along the route). `index.html`
should not need any changes. See the field reference table in README.md
for the exact shape of each anomaly object (`id`, `dist`, `type`, `depth`,
`repair`, `erf`, `lat`/`lon`/`alt`, etc.).
