***


# 🌳 Quantifying the "Invisible Forest"
### 3D Biometric Gap Analysis & Urban Digital Twin of Hong Kong's High-Density Canopy

![R](https://img.shields.io/badge/R-276DC3?style=for-the-badge&logo=r&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![LiDAR](https://img.shields.io/badge/LiDAR-Data-green?style=for-the-badge)
![HK-CSDI](https://img.shields.io/badge/HK--CSDI-Open_Data-orange?style=for-the-badge)

## 📌 Executive Summary
In the hyper-dense urban fabric of Hong Kong, a significant portion of the vegetated landscape exists as an **"Invisible Forest"**—trees that provide critical ecosystem services and pose potential structural risks but remain omitted from formal municipal registers. 

This project establishes a **Deterministic Digital Twin** to identify the biometric gap between the **Physical Reality** (detected via 2020 Airborne LiDAR) and the **Administrative Record** (Municipal Tree Registers). By focusing strictly on Parks, Roads, and Slopes managed by **LCSD, HyD, ArchSD, CEDD, and DSD**, this study seeks to isolate the "delta" of unmanaged trees.

---

## 🛠️ Technical Architecture

### 1. Spatial Foundation & Coordinate System
*   **CRS:** Hong Kong 1980 Grid (**EPSG:2326**)
*   **LiDAR Baseline:** CEDD 2020 Territory-wide Survey (.laz)
*   **Land Use Mask:** PlanD 2020 Raster Grids (`LUM_end2020.tif`)

### 2. The 4-Phase Workflow

| Stage | Focus | Primary Tools |
| :--- | :--- | :--- |
| **Stage 1** | **Jurisdictional Masking** | `sf`, `raster`, `lidR` |
| **Stage 2** | **2D AI Segmentation** | `SAM`, `YOLOv11-seg` |
| **Stage 3** | **3D Biometric Extraction** | `lidR` (Alpha-shapes, LMF) |
| **Stage 4** | **Administrative Audit** | `Geospatial Left-Join` |

---

## 🚀 Key Methodology

### 🛡️ Stage 1: The Liability Mask (Cookie-Cutter Logic)
To ensure the audit is administratively relevant, we use a multi-layered mask to exclude private land:
*   **Included:** Parks (LUM 41/42), Roads (LUM 51), Infrastructure (LUM 61), and SIMAR Slopes.
*   **Excluded:** Private Residential (LUM 1) and Public Housing (LUM 2).
*   **Filter Logic:** $Liability Zone = (Public Infrastructure \cup Slopes) \setminus Private Estates$

### 🤖 Stage 2: Instance Segmentation (SAM + YOLO)
We use a **SAM-to-YOLOv11-seg** pipeline to solve the "Cluster Problem" of interlocking broadleaf canopies (e.g., *Ficus microcarpa*). This provides pixel-perfect polygons instead of simple bounding boxes, allowing for cleaner 3D LiDAR extraction.

### 📊 Stage 3: 3D Biometrics
Using the AI-generated polygons, we "cookie-cut" the 1.7GB LiDAR tiles.
*   **Height Threshold:** $H_{min} > 5.0m$ (Deletes grass, shrubs, and small saplings).
*   **Metrics:** Maximum Height ($H_{max}$), 3D Crown Volume ($V_{crown}$), and Vertical Distribution Ratio ($VDR$).

---

## 💻 Code Snippets (Laptop-Safe LiDAR Processing)

```R
# Setup the out-of-core processing engine for large files
library(lidR)

# Initialize the catalog virtually (16GB RAM Safe)
ctg <- readLAScatalog("./data/lidar_tiles/")

# Configure the engine
opt_chunk_size(ctg)   <- 250   # 250m squares
opt_chunk_buffer(ctg) <- 30    # 30m overlap for whole crowns
opt_select(ctg)       <- "xyz" # Load only coordinates
opt_filter(ctg)       <- "-keep_class 2 3 4 5" # Ground + Veg only

# Normalize Height (Find the true height above ground)
normalized_ctg <- normalize_height(ctg, tin())
```

---

## 📂 Repository Structure
```text
├── 01_masking/             # Raster-to-Vector jurisdictional masking
├── 02_segmentation/        # SAM & YOLOv11-seg training scripts
├── 03_biometrics/          # lidR scripts for volumetric extraction
├── 04_audit/               # Statistical gap analysis (Rmd)
├── assets/                 # Legends, CRS definitions, and documentation
└── README.md
```

## 📋 Requirements
*   **R 4.0+:** `lidR`, `sf`, `raster`, `dplyr`
*   **Python 3.9+:** `ultralytics` (YOLOv11), `segment-anything`, `geopandas`
*   **Hardware:** 16GB RAM minimum (32GB recommended for LiDAR normalization).


---

## 📝 Acknowledgements
*   **Data Providers:** Hong Kong CSDI Portal & CEDD Spatial Data Portal.
*   **Technical Frameworks:** Based on the `lidR` package (Silva et al.) and the `YOLO` computer vision framework.

---
