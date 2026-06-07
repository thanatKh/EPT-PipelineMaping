# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, static web app that visualises In-Line Inspection (ILI) anomaly
data for a PTT OR 18-inch subsea pipeline (T61 to Sea Berth, 2.7 km) on an
interactive Leaflet.js map with a synchronised elevation profile. No build
step, no framework, no npm — pure HTML/CSS/JS plus one data file.

## Files

- `index.html` — the entire app: markup, CSS, and all JS logic (map setup,
  filters, sidebar, elevation chart, measurement tool, detail panels).
- `data.js` — all anomaly/route data, loaded globally via `<script src="data.js">`.
  Exports `ANOMALIES`, `DENTS`, `ROUTE`, `REFS`, `ROUTE_PROFILE`.
- `README.md` — full feature documentation, data field reference, and update
  instructions for future ILI runs. Read it before changing data handling —
  it documents every anomaly object field and the meaning of `repair`/`depth`/
  `erf` classifications.
- `logo.png` — branding image used in the top bar.

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
- **Marker rendering**: `rebuildMarkers()` is the core function that clears
  and repopulates all marker layers based on current filter state
  (`activeRepair`, `showExternal`, `showInternal`, `showDent`, `minDepth`).
  Call it whenever a filter changes. `dentShouldShow()`/`syncDentLayer()`
  handle the dent layer specifically.
- **Tooltips & detail panel**: `makeTooltip(a, col)` builds the hover
  tooltip HTML; `showDetail(a, col)` renders the full data card (wall-loss
  bar, elevation zone badge) when a marker is clicked.
- **Sidebar**: collapsible via `#sidebar`/`#sidebar-toggle`, contains all
  filter toggles and live statistics.
- **Measurement tool**: `measuring`, `measurePts`, `onMeasureClick`,
  `clearMeasure`, `stopMeasuring`, `haversine`, `formatDist` — click-two-
  points distance measurement drawn directly on the map.
- **Elevation profile**: `drawElev()` renders a canvas-based bathymetric
  profile chart from `ROUTE_PROFILE`/`ELEV_PROFILE` and `ZONES` (depth-band
  colour ranges). `toX(dist)`/`xToDist(canvasX)` convert between chainage
  and canvas pixel position using Leaflet's own projection so the chart
  stays pixel-aligned with the map viewport as it pans/zooms.
  `elevHighlight(dist, alt, col)` syncs a highlighted point between map
  and chart; `toggleElev()` shows/hides the panel.
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
