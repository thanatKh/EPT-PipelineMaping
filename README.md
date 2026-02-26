# 📍 ILI Pipeline Anomaly Interactive Map
### PTT Oil and Retail Business (OR) — Engineering & Maintenance

An interactive, high-performance web dashboard designed for visualizing **In-Line Inspection (ILI)** data. This tool allows engineers to monitor pipeline health, identify critical anomalies (corrosion, dents, clusters), and prioritize repair schedules through a modern geospatial interface.



---

## 🚀 Key Features

* **Geospatial Visualization:** Real-time mapping of pipeline routes and anomalies using Leaflet.js.
* **Urgency-Based KPIs:** Top-bar dashboard showing counts for **Critical (≤ 1 year)**, **Mid-term (5-7 years)**, and **Stable** anomalies.
* **Advanced Filtering:** * Filter by **Repair Urgency** categories.
    * Dynamic **Depth Threshold** slider (0–100%) to isolate deep corrosion.
* **Search Engine:** Instant lookup by Anomaly ID to fly the camera directly to the point of interest.
* **Detail HUD:** A floating "Heads-Up Display" providing technical specs (Type, Depth %, Orientation, and GPS) for selected points.
* **Clean Enterprise UI:** A light-themed, professional aesthetic optimized for engineering workstations.

---

## 🛠️ Technical Stack

* **Frontend:** HTML5, CSS3 (Modern Flexbox/Grid), JavaScript (ES6+).
* **Mapping Engine:** [Leaflet.js](https://leafletjs.com/) (using CartoDB Voyager light tiles).
* **Fonts:** Inter (Google Fonts).
* **Deployment:** Ready for [Render](https://render.com) or GitHub Pages.

---

## 📂 Project Structure

```text
├── index.html          # The main application file
├── README.md           # Project documentation
└── /assets             # (Optional) Logos and static images
