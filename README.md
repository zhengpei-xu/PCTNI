# Belfast Cycle Network Assessment

**Level of Traffic Stress & Low-Stress Connectivity Analysis**

> Developed for the [PCTNI Data-Driven Transport Planning Hackathon](https://pctni.github.io/hack/) (18 May 2026, Queen's University Belfast), hosted by Prof. Robin Lovelace.

**Author:** Zhengpei (Pippin) Xu — MRes Urban Spatial Science, UCL CASA

---

## 🚲 See the Results

### 📊 Interactive Dashboard — Belfast LTS Connectivity Explorer

👉 **[Launch Dashboard](https://zhengpei-xu.github.io/PCTNI/belfast-cycling-dashboard/belfast_lts_connectivity_dashboard.html)**

Explore Belfast's cycling network interactively:
- LTS-classified road network map (colour-coded by stress level)
- Click any Super Data Zone to view its LTS distribution
- Drag the detour tolerance slider and watch connectivity metrics update in real time
- Toggle between city-wide and per-SDZ analysis

> To run locally: `cd belfast-cycling-dashboard && python -m http.server 8080`, then open `http://localhost:8080/belfast_lts_connectivity_dashboard.html`

### 📝 Project Website (Quarto)

👉 **[View Full Report](https://zhengpei-xu.github.io/PCTNI/_site/index.html)**

The Quarto site includes the project overview, LTS methodology background, and full technical notebooks with code and commentary.

---

## Key Findings

| LTS Threshold | % Trips Connected | % Nodes Connected | Who Can Ride? |
|:---:|:---:|:---:|:---|
| LTS ≤ 1 | 0.0% | 0.0% | Children / all ages and abilities |
| LTS ≤ 2 | 0.7% | 0.8% | Mainstream adult population |
| LTS ≤ 3 | 7.9% | 8.3% | Confident, experienced cyclists |
| LTS ≤ 4 | 100% | 100% | All roads, no filtering |

Belfast has plenty of quiet residential streets — but they don't form a connected network. Cyclists are forced onto high-stress arterials to complete their journeys. Even at LTS ≤ 3, only 6% of commuting trips can be served without excessive detour.

---

## Overview

This project classifies every cyclable road segment in Belfast into four stress levels (LTS 1–4) based on speed limits, lane counts, and cycling infrastructure, following the Conveyal/Mekuria framework augmented by Jeong & Smith (2025). I then compute Mekuria's **Percent Trips Connected** metric using Census 2021 commuting flows to measure how much of Belfast's travel demand can actually be served by a connected, low-stress cycling network.

---

## Methodology

### 1. LTS Classification

Each cyclable road segment is assigned an LTS level (1–4) using a three-stage pipeline:

1. **Cycle infrastructure classification** — 6 categories (Protected a/b/c, Unprotected Mandatory/Advisory, Off-Road) derived from OSM tags following Jeong & Smith (2025)
2. **Conveyal per-edge classification** — speed × lane count decision matrix from Mekuria et al. (2012), with Routino highway-type fallback for missing data
3. **Intersection adjustment** — Jeong & Smith (2025) signalised junction modification, distinguishing signal-controlled from unsignalised crossings

### 2. Graph Construction (Hybrid Approach)

| Source | Strength | Weakness |
|:---|:---|:---|
| **pyrosm** (PBF extract) | Full OSM tag preservation → accurate LTS | No shared node IDs → fragmented graph (13,000+ components) |
| **OSMnx** (Overpass API) | Topological connectivity → single connected component | Incomplete tags → inaccurate LTS if classified directly |

I use OSMnx for graph topology and transfer pyrosm-derived LTS values via dense-sampling nearest-neighbour matching (100% match rate with `network_type='bike'`).

### 3. Connectivity Analysis

Following Mekuria et al. (2012):

- For each LTS threshold (1–4), extract a sub-graph of edges ≤ that threshold
- For each OD pair (Census 2021 commuting flows at SDZ level), check if a low-stress path exists
- Apply the **detour criterion**: the low-stress path must not exceed the shortest route by more than 25% (or 530 m for short trips)
- Only OD pairs with network distances between 0.5–6 miles are included

---

## Project Structure

```
belfast-cycle-network/
│
├── README.md
├── _quarto.yml                          # Quarto site config
├── index.qmd                            # Landing page
├── .nojekyll                            # Force GitHub Pages to bypass Jekyll
│
├── Code/
│   ├── lts_background.ipynb             # What is LTS? (methodology)
│   ├── belfast_cycle_lts_assessment.ipynb    # LTS classification
│   └── belfast_percent_trips_connected.ipynb # Connectivity analysis
│
├── belfast-cycling-dashboard/
│   ├── belfast_lts_connectivity_dashboard.html
│   ├── lts_network.json                 # LTS road network (map layer)
│   ├── sdz_data.json                    # SDZ boundaries + LTS counts
│   └── od_data.json                     # OD distances (detour slider)
│
├── Data/                                # Input data (not tracked in git)
│   ├── zones_sdz.gpkg
│   ├── od_ni_open_filtered.csv
│   ├── LTS_Belfast.parquet
│   ├── cycling_network_osmnx.parquet
│   └── cycling_nodes_osmnx.parquet
│
└── Figure/                              # Generated outputs
    ├── Belfast_LTS.png
    ├── belfast_connectivity_metrics.png
    ├── belfast_sdz_connectivity_choropleth.png
    └── *.png
```

---

## Data Sources

| Dataset | Source | Description |
|:---|:---|:---|
| Road network | [OpenStreetMap](https://www.openstreetmap.org/) | via pyrosm (PBF) + OSMnx |
| OD commuting flows | [PCTNI Hackathon](https://github.com/pctni/hack/releases/tag/v0.1.0) / Census 2021 ODWP01 | Workplace commuting at SDZ level |
| Zone boundaries | [PCTNI Hackathon](https://github.com/pctni/hack/releases/tag/v0.1.0) | NISRA Super Data Zones 2021 |

---

## Reproducing the Analysis

### Prerequisites

```bash
conda create -n urbsim python=3.11
pip install geopandas osmnx pyrosm igraph networkx scipy pandas matplotlib seaborn folium 
```

### Steps

1. Place `zones_sdz.gpkg` and `od_ni_open_filtered.csv` in `Data/`
2. Run `belfast_cycle_lts_assessment.ipynb` → produces `LTS_Belfast.parquet`
3. Run `belfast_percent_trips_connected.ipynb` → produces dashboard JSON files
4. Launch dashboard: `cd belfast-cycling-dashboard && python -m http.server 8080`
5. Render Quarto site: `quarto render`

---

## References

- Mekuria, M. C., Furth, P. G. & Nixon, H. (2012). *Low-Stress Bicycling and Network Connectivity*. Mineta Transportation Institute Report 11-19.
- Conveyal (2015). *Estimated Level of Traffic Stress*. https://docs.conveyal.com/learn-more/traffic-stress
- Jeong, P. & Smith, D. (2025). *Improving Infrastructure and Accessibility Indicators for Urban Cycling Networks*. CASA Working Paper 242.

---

## Licence

Code: MIT. Data: OpenStreetMap © contributors (ODbL); Census data Crown Copyright (NISRA).
