# 🛢️ PTT OR — ILI Pipeline Anomaly Map

> **Engineering & Maintenance Department**  
> PTT Oil and Retail Business Public Company Limited

An interactive web-based map for visualising **In-Line Inspection (ILI)** anomaly data from the 18″ T61 → Sea Berth pipeline (2.7 km). Built for internal engineering reviews and meeting presentations — no server or internet required to run.

---

## 📸 Overview

The app plots all **2,014 metal loss anomalies** and **1 geometric dent** on a real satellite map, colour-coded by repair urgency. Critical markers (red/orange) are always rendered on top of the green mass so they are never hidden.

| Layer | Count | Colour |
|---|---|---|
| Repair ≤ 1 year | 1 | 🔴 Red |
| Repair 5–7 years | 1 | 🟠 Orange |
| Repair 7–10 years | 1 | 🟡 Yellow |
| Repair > 10 years | 2,012 | 🟢 Green |
| Dent (geometric) | 1 | 🩷 Pink |
| Valves / Reference pts | 6 | 🟨 Gold |

---

## 🗂️ File Structure

```
📁 repository/
  ├── pipeline_map.html   # UI — map interface, filters, sidebar (~32 KB)
  └── data.js             # Data — all anomaly GPS coordinates (~607 KB)
```

**Both files must be in the same folder.**  
`data.js` is loaded by `pipeline_map.html` via `<script src="data.js"></script>`.

To update anomaly data for a future ILI run, **only edit `data.js`** — the HTML never needs to change.

---

## 🚀 Getting Started

### Option 1 — Python (recommended, zero dependencies)
```bash
cd /path/to/folder
python3 -m http.server 8080
```
Then open: [http://localhost:8080/pipeline_map.html](http://localhost:8080/pipeline_map.html)

### Option 2 — VS Code Live Server
Install the [Live Server extension](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer), right-click `pipeline_map.html` → **Open with Live Server**.

### Option 3 — Node.js
```bash
npx serve .
```

> ⚠️ **Do not open `pipeline_map.html` by double-clicking** (i.e. `file://` protocol).  
> Browsers block local `<script src="data.js">` imports for security. A local server is required.

---

## ✨ Features

| Feature | Description |
|---|---|
| **Satellite / Street map** | Toggle between Esri World Imagery and OpenStreetMap |
| **Repair priority filter** | Toggle each priority layer on/off with iOS-style switches |
| **Surface location chips** | Filter by External / Internal / Mid-wall corrosion |
| **Depth slider** | Show only anomalies ≥ N% wall thickness |
| **Live statistics** | Visible count, max depth, external/internal split — updates on every filter change |
| **Search** | Find anomaly by Feature ID, distance (m), or type |
| **Hover tooltip** | Depth bar, ERF, repair classification on hover |
| **Click detail panel** | Full 11-field data card with animated wall-loss bar |
| **Zoom & pan** | Standard Leaflet controls; marker size scales with depth % WT |
| **Collapsible sidebar** | One-click collapse to maximise the map |

---

## 🗺️ Data Source

| Field | Value |
|---|---|
| **Pipeline** | 18″ T61 to Sea Berth, 2.7 km — Code 18T64T61 |
| **Product** | Oil |
| **Inspection date** | 22–23 November 2025 |
| **Tools** | CLP+IMU (geometry) + UTWM (wall measurement) |
| **Contractor** | PIPECARE DMCC, Dubai |
| **ILI Report** | Rev. 1 — 21 Jan 2026 |
| **FFP & RRR Report** | Rev. 0 — 30 Jan 2026 |
| **Assessment standard** | API 579 / ASME B31G / ASME B31.4 / API 1160 |
| **Fitness verdict** | ✅ Fit to operate until end of 2030 |

GPS coordinates are WGS84 decimal degrees, extracted directly from the ILI Pipetally Excel report.

---

## 🔧 Updating Data for a Future Inspection

Open `data.js` — the header comment documents every field:

```js
// Each anomaly object:
{
  id:       "1168",           // Feature number
  dist:     1068.108,         // Distance from launcher (m)
  type:     "corrosion",      // corrosion | corrosion cluster | dent smooth
  dim:      "General",        // General | Circumferential grooving
  depth:    74.0,             // Depth % wall thickness
  depthMm:  9.1,              // Depth in mm
  location: "External",       // External | Internal | Middle
  erf:      0.108,            // Estimated Repair Factor (API 579)
  lat:      13.105636,        // WGS84 latitude
  lon:      100.8784186,      // WGS84 longitude
  alt:      -3.31,            // Altitude (m)
  wt:       12.39,            // Wall thickness (mm)
  axLen:    38,               // Axial length (mm)
  width:    59,               // Width (mm)
  orient:   "05:52:00",       // Clock orientation
  repair:   ">10yr",          // <=1yr | 5-7yr | 7-10yr | >10yr | monitored
  comment:  ""
}
```

Replace the `ANOMALIES`, `DENTS`, `ROUTE`, and `REFS` arrays in `data.js` with data from the new Pipetally Excel export. The HTML requires no changes.

---

## 🛠️ Tech Stack

| Library | Version | Purpose |
|---|---|---|
| [Leaflet.js](https://leafletjs.com/) | 1.9.4 | Map rendering, markers, tooltips |
| [Esri World Imagery](https://www.arcgis.com/) | — | Satellite tile layer |
| [OpenStreetMap](https://www.openstreetmap.org/) | — | Street tile layer |
| [DM Sans](https://fonts.google.com/specimen/DM+Sans) | — | UI typography |

No build tools, no frameworks, no npm. Pure HTML + CSS + JS.

---

## 📋 Pipeline Quick Reference

```
Owner       : PTT Oil and Retail Business Public Company Limited
Code        : 18T64T61
Route       : T61 Station → Sea Berth
Length      : 2.7 km
Diameter    : 18 inch (API 5L Gr B, longitudinal welded)
Wall Thick. : 9.5 mm nominal
Installed   : 1975 (50 years at time of inspection)
MAOP        : 10.3 Bar
Design P.   : 14.3 Bar
Previous ILI: 2012 (13 years prior)
ERF ≥ 1.0   : 0 anomalies
Max depth   : 74% WT (external, at 1,068 m)
```

---

## 📄 License

Proprietary — PTT Oil and Retail Business Public Company Limited.  
For internal use by the Engineering & Maintenance Department only.
