# NDWI-Based Annual Water Extent Monitoring — Brahmaputra River (2019–2024)

A Google Earth Engine + Python case study deriving annual surface water indicators for the **Brahmaputra River basin** from Sentinel-2 imagery, using the Normalized Difference Water Index (NDWI) with per-pixel Cloud Score+ masking.

This notebook is part of an ongoing geospatial Python toolkit covering rainfall trend analysis, land-cover classification, terrain visualization, and multi-temporal change detection for river/hazard-relevant AOIs.

---

## Why the Brahmaputra

The Brahmaputra is one of the world's major transboundary rivers, flowing through Tibet (China), India, and Bangladesh. It is defined by an exceptionally dynamic channel morphology — extensive braiding, rapid seasonal channel migration, and among the highest sediment loads of any river system globally. Combined with intense monsoon-driven discharge variability, this makes consistent, remote-sensing-based water extent monitoring especially valuable for:

- **Flood hazard characterization** — annual water occurrence baselines against which anomalous inundation can be measured
- **Channel dynamics** — quantifying braided-channel migration, sandbar (char) formation/erosion, and bank-line retreat
- **Exposure/vulnerability mapping** — separating persistent water bodies from seasonally inundated floodplain

---

## What this notebook does

For each year in the study period (2019–2024), the pipeline produces three co-registered raster layers from Sentinel-2 SR Harmonized imagery:

| Layer | Description |
|---|---|
| `NDWI_yearly_median` | Continuous, per-pixel median NDWI across all clear-sky observations in the year (McFeeters, 1996) |
| `NDWI_water_frequency` | Fraction (0–1) of clear-sky observations in which each pixel was classified as water |
| `NDWI_yearly_binary` | Persistent water mask — pixels classified as water in >50% of clear observations, distinguishing stable water bodies from transient/seasonal wet surfaces |

### Pipeline steps

1. **AOI setup** — river basin boundary loaded from a shapefile (`brahmaputra.shp`, included in this repo), reprojected to `EPSG:4326`, and converted to an `ee.Geometry` via `geemap.geopandas_to_ee()`.
2. **Cloud/shadow masking** — per-pixel masking using [Cloud Score+](https://developers.google.com/earth-engine/datasets/catalog/GOOGLE_CLOUD_SCORE_PLUS_V1_S2_HARMONIZED) (`cs` band, threshold 0.6), linked to the Sentinel-2 collection via `linkCollection()`.
3. **Index calculation** — NDVI and NDWI computed per clear-sky pixel; NDWI thresholded at 0.0 (standard McFeeters cutoff) to derive a binary water flag per image.
4. **Annual composites** — median, occurrence-frequency, and persistence-thresholded binary layers built per year and clipped to the true basin polygon.
5. **Visualization** — interactive `geemap` viewer with a year-selection dropdown (median / frequency / persistent-water layers toggleable in the layer panel), plus static `xarray`-based faceted subplots for publication-style multi-year comparison.
6. **Export** — full multi-year stack pulled into a local `xarray.Dataset` via `xee`, saved as NetCDF for offline reuse without re-querying Earth Engine.

---

## Data

- **Sentinel-2 SR Harmonized** (`COPERNICUS/S2_SR_HARMONIZED`) — 10 m Green/Red/NIR bands
- **Cloud Score+** (`GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED`) — per-pixel clear-sky probability
- **Brahmaputra river basin boundary** — `brahmaputra.shp` (included in this repository under `/data`)

---

## Repository structure

```
├── Python_for_GIS_NDWI_case_study_Brahmaputra.ipynb   # main notebook
├── data/
│   └── brahmaputra.shp (+ .shx, .dbf, .prj, ...)       # river basin shapefile (AOI)
└── README.md
```

> **Note:** the shapefile's full component set (`.shp`, `.shx`, `.dbf`, `.prj`, `.cpg`) must all be present in `/data` for `geopandas.read_file()` to load it correctly.

---

## Requirements

```
earthengine-api
geemap
geopandas
xarray
xee
rioxarray
matplotlib
seaborn
numpy
pandas
ipywidgets
requests
```

Earth Engine access requires a registered [Google Earth Engine account](https://earthengine.google.com/) and `ee.Authenticate()` on first run.

---

## Usage

1. Clone this repository.
2. Update the shapefile path in the notebook to point at `data/brahmaputra.shp`.
3. Run `ee.Authenticate()` / `ee.Initialize()` (high-volume endpoint recommended for repeated composite generation).
4. Adjust `years = list(range(2019, 2025))` to your desired study window.
5. Run all cells — the interactive map and static comparison figures will render inline; the NetCDF export cell is optional and only needed if you want an offline copy of the pixel data.

---

## Key references

- McFeeters, S. K. (1996). The use of the Normalized Difference Water Index (NDWI) in the delineation of open water features. *International Journal of Remote Sensing*, 17(7), 1425–1432.
- Pekel, J.-F., Cottam, A., Gorelick, N., & Belward, A. S. (2016). High-resolution mapping of global surface water and its long-term changes. *Nature*, 540(7633), 418–422.
- Gorelick, N., et al. (2017). Google Earth Engine: Planetary-scale geospatial analysis for everyone. *Remote Sensing of Environment*, 202, 18–27.
- Pasquarella, V. J., et al. (2023). Cloud Score+: A cloud, cloud shadow, and cirrus/haze masking algorithm for Sentinel-2. Google Research.

---
