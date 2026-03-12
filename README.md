# PTT OR — ILI Pipeline Anomaly Map

**Engineering & Maintenance Department**  
PTT Oil and Retail Business Public Company Limited

An interactive web-based map for visualising In-Line Inspection (ILI) anomaly data from the 18-inch T61 to Sea Berth pipeline (2.7 km). Built for internal engineering reviews and meeting presentations — no server or internet connection required beyond the initial tile load.

---

## Overview

The app plots all **2,014 metal loss anomalies** and **1 geometric dent** on a real satellite map, colour-coded by repair urgency. Critical markers are always rendered on top so they are never hidden beneath the green mass. An elevation profile panel below the map shows the pipeline's bathymetric profile, synchronised in real time with the map's current viewport.

| Layer | Count | Colour |
|---|---|---|
| Repair <= 1 year | 1 | Red |
| Repair 5-7 years | 1 | Orange |
| Repair 7-10 years | 1 | Yellow |
| Repair > 10 years | 2,012 | Green |
| Dent (geometric) | 1 | Pink |
| Valves / Reference points | 6 | Gold |

---

## File Structure

```
repository/
  pipeline_map.html   -- UI: map interface, filters, sidebar, elevation chart
  data.js             -- Data: all anomaly GPS coordinates and route profile (~635 KB)
  README.md           -- This file
```

Both files must be in the same folder. `data.js` is loaded by `pipeline_map.html` via `<script src="data.js"></script>`.

To update anomaly data for a future ILI run, **only edit `data.js`** — the HTML never needs to change.

---

## Getting Started

### Option 1 — Python (recommended, zero dependencies)
```bash
cd /path/to/folder
python3 -m http.server 8080
```
Then open: http://localhost:8080/pipeline_map.html

### Option 2 — VS Code Live Server
Install the Live Server extension, right-click `pipeline_map.html`, then select Open with Live Server.

### Option 3 — Node.js
```bash
npx serve .
```

**Important:** Do not open `pipeline_map.html` by double-clicking (file:// protocol). Browsers block local `<script src="data.js">` imports for security reasons. A local HTTP server is required.

---

## Features

### Map View

| Feature | Description |
|---|---|
| Satellite / Street map | Toggle between Esri World Imagery and OpenStreetMap |
| Repair priority filter | Toggle each priority layer on/off independently |
| Defect type filter | Filter by External corrosion, Internal corrosion, or Dent |
| Depth slider | Show only anomalies at or above a chosen wall thickness percentage |
| Live statistics | Visible count, max depth, and external/internal split — updates on every filter change |
| Search | Find anomaly by Feature ID, chainage (m), or defect type |
| Hover tooltip | Depth bar, ERF, and repair classification on hover |
| Click detail panel | Full data card with animated wall-loss bar and elevation zone badge |
| Measurement tool | Click two points on the map to measure straight-line distance |
| Collapsible sidebar | One-click collapse to maximise the map area |

### Elevation Profile

| Feature | Description |
|---|---|
| Synchronised viewport | Profile X-axis tracks the map in real time — pan or zoom the map and the profile updates instantly |
| True pixel projection | X-axis positions are derived from Leaflet's own map projection, giving geometrically exact alignment between the map route and the profile curve |
| Bathymetric zone bands | Colour-coded depth bands: above sea level, 0 to -5 m, -5 to -10 m, -10 to -15 m, -15 to -20 m, below -20 m |
| Anomaly dots | Coloured by repair priority; respects all active filters |
| Hover tooltip | Shows Feature ID, chainage, altitude, zone, and depth % WT for visible (filtered) anomalies only |
| Click to detail | Click any anomaly dot to open its full data card and pan the map to its location |
| Open by default | Profile panel is shown on load; can be toggled with the button above it |

### Legend

The Anomaly Legend in the bottom-right corner of the map can be collapsed by clicking the title bar, and expanded again the same way.

---

## Data Sources

| Field | Value |
|---|---|
| Pipeline | 18-inch T61 to Sea Berth, 2.7 km — Code 18T64T61 |
| Product | Fuel Oil |
| Inspection date | 22-23 November 2025 |
| Tools | CLP+IMU (geometry) and UTWM (ultrasonic wall measurement) |
| Contractor | PIPECARE DMCC, Dubai |
| ILI Report | Rev. 1 — 21 January 2026 |
| FFP and RRR Report | Rev. 0 — 30 January 2026 |
| Assessment standard | API 579 / ASME B31G / ASME B31.4 / API 1160 |
| Fitness verdict | Fit to operate until end of 2030 |

GPS coordinates are WGS84 decimal degrees, extracted directly from the ILI Pipetally Excel report. Altitudes are from the IMU pig run.

---

## Contents of data.js

`data.js` exports five JavaScript constants loaded globally by the HTML:

| Constant | Description |
|---|---|
| `ANOMALIES` | Array of 2,014 metal loss anomaly objects |
| `DENTS` | Array of 1 geometric dent object |
| `ROUTE` | Array of 207 GPS waypoints defining the pipeline centreline |
| `REFS` | Array of reference point markers (valves, launchers, receivers) |
| `ROUTE_PROFILE` | Array of 206 route points with altitude — used for the elevation profile chart |

`ROUTE_PROFILE` was added in the current revision. It is derived from `ROUTE` (lat/lon/dist) with altitude values interpolated from the `ANOMALIES` dataset. All other constants are unchanged from the original Pipetally export.

### Anomaly object fields

```js
{
  id:       "1168",           // Feature number
  dist:     1068.108,         // Chainage from launcher (m)
  type:     "corrosion",      // corrosion | corrosion cluster | dent smooth
  dim:      "General",        // General | Circumferential grooving
  depth:    74.0,             // Depth % wall thickness
  depthMm:  9.1,              // Depth in mm
  location: "External",       // External | Internal | Middle
  erf:      0.108,            // Estimated Repair Factor (API 579)
  lat:      13.105636,        // WGS84 latitude
  lon:      100.8784186,      // WGS84 longitude
  alt:      -3.31,            // Altitude (m, from IMU)
  wt:       12.39,            // Measured wall thickness (mm)
  axLen:    38,               // Axial length (mm)
  width:    59,               // Circumferential width (mm)
  orient:   "05:52:00",       // Clock orientation
  repair:   ">10yr",          // <=1yr | 5-7yr | 7-10yr | >10yr | monitored
  comment:  ""
}
```

To update for a future inspection, replace `ANOMALIES`, `DENTS`, `ROUTE`, and `REFS` with data from the new Pipetally export. `ROUTE_PROFILE` must also be regenerated from the new route and altitude data. The HTML requires no changes.

---

## Tech Stack

| Library | Version | Purpose |
|---|---|---|
| Leaflet.js | 1.9.4 | Map rendering, markers, tooltips, coordinate projection |
| Esri World Imagery | -- | Satellite tile layer |
| OpenStreetMap | -- | Street map tile layer |
| DM Sans (Google Fonts) | -- | UI typography |

No build tools, no frameworks, no npm. Pure HTML, CSS, and JavaScript.

---

## Pipeline Quick Reference

```
Owner          : PTT Oil and Retail Business Public Company Limited
Code           : 18T64T61
Route          : T61 Station to Sea Berth
Length         : ~2.7 km
Diameter       : 18 inch (API 5L Gr B, longitudinal welded)
Wall thickness : 9.5 mm nominal
Installed      : 1975 (50 years at time of inspection)
MAOP           : 10.3 Bar
Design P.      : 14.3 Bar
Previous ILI   : 2012 (13 years prior)
ERF >= 1.0     : 0 anomalies
Max depth      : 74% WT (external corrosion, at chainage 1,068 m)
```

---

## License

Proprietary — PTT Oil and Retail Business Public Company Limited.  
For internal use by the Engineering and Maintenance Department only.
