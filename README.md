## `forestseg-als`

**Advanced Tree Segmentation and Volume Estimation from Airborne LiDAR Data**

---

## Table of Contents

* Project overview
* Objectives
* Workflow pipeline
* Features
* Architecture and core algorithms
* Input and output data
* Volume estimation methodology
* Testing and validation with LiDAR360 v9 beta
* Setup and usage
* Folder structure
* Future development
* License

---

## Project Overview

`forestseg-als` is a research-grade processing framework designed to segment and analyze **individual trees** from **Airborne Laser Scanning (ALS)** data.
The project focuses on generating **accurate tree-level outputs**—including stem locations, crown boundaries, and estimated tree volumes—through voxelization, clustering, and 3D traversal techniques.

This project was developed and tested in collaboration with **GreenValley International**, supporting **LiDAR360 v9 beta** forestry module testing and model accuracy validation.

---

## Objectives

* Extract individual trees from dense ALS point clouds.
* Evaluate the performance of LiDAR360 v9 beta forestry segmentation models.
* Produce accurate per-tree volume estimates using derived structural parameters.
* Provide a reproducible, open framework for testing alternative segmentation and volume algorithms.

---

## Workflow Pipeline

**Input:** Airborne LiDAR (ALS) point cloud (LAS/LAZ format)

**Processing Stages:**

1. **Voxelization** – Discretize the 3D space into uniform volumetric cells for efficient spatial analysis.
2. **Clustering** – Apply spatial and height-based clustering (e.g., DBSCAN or region-growing) to separate major canopy groups.
3. **Vertical Traversing** – Identify tree trunks and vertical continuity within clusters to distinguish overlapping crowns.
4. **Individualization** – Isolate single trees from canopy clusters using structural continuity and height gradients.
5. **Feature Extraction** – Derive metrics such as tree height, crown diameter, and stem position.
6. **Volume Calculation** – Estimate tree volume using allometric equations and derived geometric parameters.
7. **Output Generation** – Export results as segmented point clouds, GeoJSON features, and tabular reports.

**Output:**

* Individualized tree point clouds
* Tree metrics table (ID, height, crown width, stem diameter, volume)
* Optional rasterized canopy height and density layers

---

## Features

* Works with raw ALS data (LAS/LAZ)
* 3D voxel grid for efficient neighborhood computation
* Multi-layer clustering (ground-separated and canopy-based)
* Tree individualization using vertical traversing of voxel clusters
* Stem and crown boundary estimation
* Volume estimation via geometric and empirical formulas
* Export to LAS, CSV, and GeoJSON
* Modular Python-based architecture for research and testing
* Compatible with LiDAR360 data formats and workflows

---

## Architecture and Core Algorithms

### 1. Voxelization

Converts the unstructured 3D point cloud into discrete cubic elements (voxels).
Each voxel stores attributes such as:

* Mean height (Z)
* Point count
* Reflectance/intensity
* Canopy class indicator

This step reduces computational complexity and allows spatial connectivity analysis.

```python
voxel_size = 0.25  # meters
voxel_grid = create_voxel_grid(point_cloud, voxel_size)
```

---

### 2. Clustering

Spatial clusters are detected using 3D connectivity (adjacent voxels within defined distance).
Each cluster approximates a canopy or tree crown region.

Common algorithms:

* **3D DBSCAN**
* **Euclidean region growing**
* **Graph-based connected components**

```python
clusters = cluster_voxels(voxel_grid, method='dbscan', eps=1.0, min_samples=10)
```

---

### 3. Vertical Traversing

Each cluster is traversed vertically (Z-direction) to detect continuous trunk structures and isolate overlapping crowns.
A vertical path search algorithm scans upward voxel stacks to determine if a crown belongs to a single or multiple trees.

```python
trees = vertical_traverse(clusters, min_height=2.0)
```

---

### 4. Individualization

Final step merges trunk paths and crown clusters into distinct tree objects.
Spatial overlap resolution uses crown shape analysis and height gradients.

---

### 5. Volume Calculation

Once individual trees are extracted, their **volume** is estimated using one of the following:

**a. Geometric Volume (from point cloud)**
Approximating the trunk as a set of conical frustums:

```python
V = (π * h / 3) * (r1² + r1*r2 + r2²)
```

**b. Allometric Equations (empirical models)**
For forest type *i*:

```python
V_i = a_i * (DBH^b_i) * (H^c_i)
```

where

* DBH = Diameter at Breast Height (m)
* H = Tree height (m)
* a_i, b_i, c_i = species/region-specific constants

Both methods can be applied and compared for validation.

---

## Input and Output Data

### Input

* **ALS data:** `.las` / `.laz`
* **Digital Terrain Model (optional):** `.tif`
* **Forest type metadata (optional)** for species-specific models

### Output

* `individual_trees.las` – segmented per-tree point clouds
* `tree_metrics.csv` – ID, height, DBH, crown diameter, volume
* `tree_polygons.geojson` – crown footprint polygons
* `report_summary.pdf` – summary statistics (optional)

---

## Testing and Validation with LiDAR360 v9 Beta

This segmentation pipeline was used to validate the **Forestry Model** module in **LiDAR360 v9 beta (GreenValley International)**.

**Objectives of Testing:**

* Evaluate accuracy of individual tree detection (precision/recall).
* Compare volume estimation results between LiDAR360’s built-in models and custom voxel-based results.
* Benchmark runtime performance for various forest densities (sparse vs dense canopy).

Testing datasets were processed through both the LiDAR360 v9 beta and `forestseg-als` pipeline, with results compared for:

* Segmentation accuracy (IoU and tree count)
* Height & volume correlation (R² and RMSE)
* Processing time efficiency

---

## Setup and Usage

### Requirements

* Python ≥ 3.10
* PDAL ≥ 2.6
* NumPy, SciPy, Open3D, scikit-learn, laspy, geopandas, rasterio, matplotlib
* Optional: LiDAR360 output data for cross-validation

### Installation

```bash
git clone https://github.com/your-org/forestseg-als.git
cd forestseg-als
pip install -r requirements.txt
```

### Example Usage

```bash
python main.py --input data/forest_als.las --voxel 0.25 --method dbscan
```

This command runs:

1. Voxelization (0.25 m grid)
2. Clustering (DBSCAN)
3. Vertical traversing
4. Individualization
5. Exports results to `/output` directory

---

## Future Development

* Integration with **LiDAR360 SDK / API** for streamlined data exchange.
* Species classification using spectral indices or point intensity.
* Integration with **Deep Learning** for canopy crown delineation.
* Batch processing for large ALS datasets.
* Visualization dashboard using Potree / Open3D / CesiumJS.

---

## License

MIT License (or specify GreenValley internal collaboration license if proprietary)

