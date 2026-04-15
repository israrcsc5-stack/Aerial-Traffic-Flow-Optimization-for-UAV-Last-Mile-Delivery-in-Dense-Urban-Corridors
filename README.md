# 🚁 UAV Last-Mile Delivery: Spatio-Temporal Occupancy-Aware Path Planning

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![Kaggle Dataset](https://kaggle.com/static/images/open-in-kaggle.svg)](https://www.kaggle.com/datasets/israrahmad45/last-mile-delivery-dataset-for-uav)
> **📄 Manuscript Notice**: This repository contains the official implementation accompanying the manuscript *"Aerial Traffic Flow Optimization for UAV Last-Mile Delivery in Dense Urban Corridors: A Spatio-Temporal Occupancy-Aware Path Planning Framework for Smart Cities"* submitted to **The Visual Computer**.  
> **If you use this code or data in your research, please cite our manuscript** (citation details below).

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Contributions](#-key-contributions)
- [Prerequisites & Installation](#-prerequisites--installation)
- [Data Sources](#-data-sources)
- [Quick Start](#-quick-start)
- [Configuration & Parameters](#-configuration--parameters)
- [Core Algorithms](#-core-algorithms)
- [Running Experiments](#-running-experiments)
- [Visualization Outputs](#-visualization-outputs)
- [Results & Reproducibility](#-results--reproducibility)
- [Citation](#-citation)
- [License](#-license)
- [Contact](#-contact)

---

## 🔍 Overview

This repository implements a **spatio-temporal occupancy-aware path planning framework** for optimizing UAV last-mile delivery in dense urban environments. The system:

1. **Extracts urban geospatial data** from OpenStreetMap (OSM) for Tokyo and Shenzhen
2. **Classifies delivery zones** into three types: `rooftop`, `ground_open`, and `constrained_corridor`
3. **Computes ST-occupancy scores** (π·r²·τ) to quantify spatio-temporal exclusion zones
4. **Simulates UAV fleet operations** using discrete-event simulation (SimPy)
5. **Evaluates three routing strategies**: 
   - `low_occ_first`: Prioritize low spatio-temporal occupancy tasks
   - `high_occ_first`: Prioritize high occupancy tasks
   - `nearest_first`: Prioritize proximity to depot
6. **Generates comprehensive visualizations** and statistical summaries for analysis

The framework enables reproducible evaluation of UAV traffic management strategies under realistic urban constraints.

---

## ✨ Key Contributions

| Component | Description |
|-----------|-------------|
| **Zone Classification Engine** | Rule-based classifier assigning delivery points to `rooftop`, `ground_open`, or `constrained_corridor` based on building height, open space proximity, and road width |
| **ST-Occupancy Scoring** | Quantifies landing zone contention as `π × exclusion_radius² × occupancy_duration` |
| **Discrete-Event Simulator** | SimPy-based engine modeling UAV queuing, travel, hovering, and zone occupancy conflicts |
| **Multi-Strategy Evaluation** | Comparative analysis of task prioritization heuristics across fleet sizes (12/24/50 UAVs) |
| **Reproducible Pipeline** | End-to-end workflow from OSM data extraction to publication-ready figures |

---

## ⚙️ Prerequisites & Installation

### Option 1: Conda (Recommended)

```bash
# Clone repository
git clone https://github.com/yourusername/uav_smart_city.git
cd uav_smart_city

# Create and activate environment
conda env create -f environment.yml
conda activate uav_rs

# Verify installation
python -c "import geopandas, simpy, rasterio; print('✓ Dependencies OK')"
```

### Option 2: Pip

```bash
python -m venv uav_env
source uav_env/bin/activate  # Linux/Mac; or uav_env\Scripts\activate on Windows

pip install -r requirements.txt
```

### `requirements.txt`
```txt
geopandas>=0.14.0
osmnx>=1.8.0
shapely>=2.0.0
fiona>=1.9.0
pyproj>=3.6.0
rasterio>=1.3.0
numpy>=1.24.0
pandas>=2.0.0
matplotlib>=3.7.0
simpy>=4.1.0
scipy>=1.11.0
scikit-learn>=1.3.0
jupyter>=1.0.0
networkx>=3.1.0
```

---

## 🌐 Data Sources (100% Free & Open)

| Source | Data Type | Resolution | Coverage | Access |
|--------|-----------|------------|----------|--------|
| **OpenStreetMap (OSM)** | Buildings, roads, open spaces | Vector | Global | [osm.org](https://www.openstreetmap.org) + Overpass API |
| **Copernicus Open Access Hub** | Sentinel-2 multispectral imagery | 10m | Global | [scihub.copernicus.eu](https://scihub.copernicus.eu) |
| **JAXA ALOS DSM** | Digital Surface Model | 30m | Japan | [earth.jaxa.jp](https://earth.jaxa.jp) |
| **SRTM / Copernicus DEM** | Elevation data | 30m | Global | [opentopography.org](https://opentopography.org) |

> ℹ️ **Pre-extracted data**: If you prefer not to download OSM data programmatically, All extracted and classified data for Tokyo and Shenzhen (building footprints, road networks, open spaces, and delivery tasks with ST-occupancy scores) are available for direct download:  
> 🔗 [Last-Mile Delivery Dataset for UAV on Kaggle](https://www.kaggle.com/datasets/israrahmad45/last-mile-delivery-dataset-for-uav)  
> *No OSM API calls required — ready for immediate simulation and analysis.*

> If you need the pre-processed data, please request the pre-processed `.gpkg` files for Tokyo/Shenzhen by emailing **israrcsc5@gmail.com** with subject: *"UAV Delivery Data Request"*.
---

## 🚀 Quick Start

### Step 1: Extract OSM Data
```bash
python src/data_collection.py --city tokyo --city shenzhen
```
*Outputs*: `data/tokyo/*.gpkg`, `data/shenzhen/*.gpkg`

### Step 2: Generate Delivery Tasks & Classify Zones
```bash
python -c "
from src.zone_classifier import generate_and_classify
generate_and_classify(city='tokyo', n_points=100, seed=42)
generate_and_classify(city='shenzhen', n_points=100, seed=42)
"
```
*Outputs*: `data/*/delivery_tasks.csv` with columns:  
`x_m, y_m, zone_type, exclusion_radius_m, occupancy_duration_s, st_occupancy_score`

### Step 3: Run Simulations
```bash
python src/simulator.py --cities tokyo shenzhen \
                        --strategies low_occ_first high_occ_first nearest_first \
                        --fleets 12 24 50 \
                        --seeds 10
```
*Outputs*: `results/simulation_summary.csv`, `results/all_results.csv`

### Step 4: Generate Visualizations
```bash
python src/visualization.py --input results/simulation_summary.csv --output figures/
```
*Outputs*: 13+ publication-ready PNG figures in `figures/`

### One-Command Full Pipeline
```bash
bash run_full_pipeline.sh  # Executes Steps 1-4 sequentially
```

---

## ⚙️ Configuration & Parameters

Edit `config.yaml` to customize:

```yaml
# UAV Physics
uav_speed_ms: 15.0          # Cruise speed (m/s)

# Zone Classification Rules
building_height_threshold_m: 15.0   # Rooftop zone criterion
open_space_proximity_m: 30.0        # Ground_open proximity threshold
open_space_min_area_m2: 500.0       # Minimum open space size
narrow_road_width_m: 8.0            # Constrained corridor road width cutoff

# Exclusion Zone Parameters (from manuscript Table 2)
zone_params:
  rooftop:              {exclusion_radius_m: 15, occupancy_duration_s: 30}
  ground_open:          {exclusion_radius_m: 25, occupancy_duration_s: 20}
  constrained_corridor: {exclusion_radius_m: 40, occupancy_duration_s: 50}

# Simulation
default_seed: 42
n_seeds: 10               # Monte Carlo repetitions for statistical robustness
```

---

## 🧠 Core Algorithms

### 1. Zone Classification (`src/zone_classifier.py`)
```python
def classify_delivery_zones(delivery_gdf, buildings_gdf, roads_gdf, open_spaces_gdf, crs_epsg):
    """
    Assigns each delivery point to one of three zone types:
    
    1. 'rooftop' (Type A): 
       - Point within 5m of building AND building height > 15m
       - Exclusion: 15m radius, 30s occupancy
    
    2. 'ground_open' (Type B): 
       - Point within 30m of open space > 500m²
       - Exclusion: 25m radius, 20s occupancy
    
    3. 'constrained_corridor' (Type C): 
       - Point within 15m of road < 8m wide AND within 20m of buildings
       - Exclusion: 40m radius, 50s occupancy
    
    Returns GeoDataFrame with added columns:
    - zone_type, exclusion_radius_m, occupancy_duration_s, st_occupancy_score
    """
```

### 2. ST-Occupancy Score
```python
# Spatio-Temporal Occupancy = Area × Time
st_occupancy_score = π × (exclusion_radius_m)² × occupancy_duration_s
```
*Interpretation*: Higher scores indicate greater potential for UAV contention; used for `low_occ_first` prioritization.

### 3. Discrete-Event Simulation (`src/simulator.py`)
Key simulation events per UAV task:
1. **Queue**: Wait for available UAV from fleet pool
2. **Travel**: Fly to delivery coordinate at `UAV_SPEED_MS`
3. **Hover**: Wait if exclusion zone occupied by another UAV (checked every 2s)
4. **Occupy**: Land and hold exclusion zone for `occupancy_duration_s`
5. **Return**: Fly back to depot

Conflict resolution: First-come-first-served with spatial exclusion checks.

### 4. Routing Strategies (`src/strategies.py`)
| Strategy | Sorting Key | Rationale |
|----------|------------|-----------|
| `low_occ_first` | `st_occupancy_score` (ascending) | Minimize contention by handling "easy" tasks first |
| `high_occ_first` | `st_occupancy_score` (descending) | Prioritize high-value/high-risk deliveries |
| `nearest_first` | `dist_to_depot` (ascending) | Minimize travel time (classic VRP heuristic) |

---

## 🧪 Running Experiments

### Reproduce Manuscript Results
```bash
# Full reproduction (≈2 minutes on 8-core CPU)
python reproduce_manuscript.py
```

### Custom Experiment
```bash
python src/simulator.py \
  --city tokyo \
  --strategy low_occ_first \
  --fleet-size 24 \
  --seeds 5 \
  --output results/custom_run.csv
```

### Parameter Sweep Example
```python
# Example: Evaluate impact of exclusion radius scaling
for scale in [0.8, 1.0, 1.2]:
    config['zone_params']['rooftop']['exclusion_radius_m'] = int(15 * scale)
    results = run_simulation(tasks_df, n_uavs=24, strategy='low_occ_first')
    print(f"Scale {scale}: Total time = {results['total_time']:.1f}s")
```

---

## 📊 Visualization Outputs

| Figure | Description | Key Insight |
|--------|-------------|-------------|
| `*_zone_map.png` | Geographic distribution of delivery zones | Shenzhen has 32% rooftop deliveries vs. Tokyo's 5% |
| `faceted_line_*.png` | Total time vs. fleet size (3 strategies) | Diminishing returns beyond 24 UAVs |
| `sankey_v2.png` | Time budget decomposition | Queue wait dominates in constrained corridors |
| `radar_chart.png` | Multi-metric strategy comparison | `low_occ_first` minimizes hover time variance |
| `scatter_matrix.png` | Distance vs. delivery components | Hover time correlates with ST-occupancy score |
| `*_network_clean.png` | Delivery network graph | Proximity edges reveal natural clustering |
| `paper_summary_figure.png` | 4-panel manuscript figure | Ready for direct inclusion in LaTeX |

All figures exported at 150-200 DPI with manuscript-ready styling (fonts, colors, legends).

---

## 🔁 Results & Reproducibility

### Key Findings (from `results/tables.txt`)
```
TABLE 1: MEAN TOTAL DELIVERY TIME (seconds) — mean ± std across 10 seeds
Fleet size                                12             24             50
City     Strategy                                                         
Shenzhen Low occupancy first   4275.2 ± 45.9  2324.3 ± 43.4  1379.5 ± 68.7
Tokyo    Low occupancy first   5093.9 ± 67.5  2829.4 ± 47.4  1625.9 ± 42.4

TABLE 2: SAVINGS vs NAIVE BASELINE (%)
Fleet size                       12    24    50
City     Strategy                              
Shenzhen Low occupancy first  91.10 95.20 97.10
Tokyo    Low occupancy first  91.00 95.00 97.10
```

### Reproducibility Checklist
- ✅ **Random seeds**: All stochastic operations use explicit `seed` parameter
- ✅ **Version pinning**: `environment.yml` locks all dependency versions
- ✅ **Deterministic OSM queries**: Overpass API calls use fixed timestamps
- ✅ **Metric CRS handling**: All spatial operations use projected coordinates (EPSG:6677 for Tokyo, EPSG:32650 for Shenzhen)
- ✅ **Statistical robustness**: Results averaged over 10 independent seeds

### Docker Support (Optional)
```bash
# Build container
docker build -t uav-delivery:latest .

# Run pipeline in container
docker run --rm -v $(pwd)/results:/app/results uav-delivery:latest \
  python reproduce_manuscript.py
```

---

## 📚 Citation

If you use this code, data, or methodology in your research, **please cite our manuscript**:

```bibtex
@article{ahmad2026uav,
  title={Aerial Traffic Flow Optimization for UAV Last-Mile Delivery in Dense Urban Corridors: A Spatio-Temporal Occupancy-Aware Path Planning Framework for Smart Cities},
  author={Ahmad, Israr and Shang, Fengjun and [Co-authors]},
  journal={The Visual Computer},
  year={2026},
  note={Manuscript submitted for publication},
  doi={10.1007/s00371-026-XXXX-X},
  url={https://github.com/yourusername/uav_smart_city}
}
```

**Permanent Archive**:  
🔗 Code DOI: `10.5281/zenodo.XXXXXXX` (update after Zenodo deposit)  
🔗 Data DOI: `10.5281/zenodo.YYYYYYY` (update after data deposit)

> 🙏 **Reminder**: This code is directly associated with the manuscript submitted to *The Visual Computer*. Proper citation enables reproducibility and acknowledges the authors' contributions to open science.

---

## 📜 License

Distributed under the **MIT License**. See `LICENSE` for details.

```text
MIT License

Copyright (c) 2026 Israr Ahmad et al.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 📬 Contact

- **Author**: Israr Ahmad  
- **Email**: israr@buaa.edu.cn, l202110005@stu.cqupt.edu.cn  
- **Affiliation**: School of Computer Science and Technology, Chongqing University of Posts and Telecommunications (CQUPT)  
- **Project Page**: UAV-Path Planning  

*For data requests, manuscript inquiries, or collaboration opportunities, please contact the first author.*

---

> ⚠️ **Disclaimer**: This software is for research purposes only. UAV operations must comply with local aviation regulations, airspace restrictions, and privacy laws. The authors assume no liability for real-world deployment.
