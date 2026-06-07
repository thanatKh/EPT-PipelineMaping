# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, static web app that visualises In-Line Inspection (ILI) anomaly
data for a PTT OR 18-inch subsea pipeline (T61 to Sea Berth, 2.7 km) on an
interactive Leaflet.js map with a synchronised elevation profile. No build
step, no framework, no npm — pure HTML/CSS/JS plus one data file. Fully
self-hosted: Leaflet and the UI font are vendored locally, so the app keeps
working offline except for the live satellite/street map tile imagery.

## Files

- `index.html` — the entire app: markup, CSS, and all JS logic (map setup,
  filters, sidebar, elevation chart, measurement tool, detail panels, CSV
  export).
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
  bar, elevation zone badge) when a marker is clicked.
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

## Updating data for a new ILI run

Per the README: only `data.js` needs to change — replace `ANOMALIES`,
`DENTS`, `ROUTE`, and `REFS` from the new Pipetally export, and regenerate
`ROUTE_PROFILE` (interpolated altitude along the route). `index.html`
should not need any changes. See the field reference table in README.md
for the exact shape of each anomaly object (`id`, `dist`, `type`, `depth`,
`repair`, `erf`, `lat`/`lon`/`alt`, etc.).
