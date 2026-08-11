An Adaptive Density Unsupervised Point Cloud Instance Segmentation Framework Based on Improved DBSCAN in Autonomous Driving Scenarios

---

📄 Paper Information

| Item | Details |
|------|---------|
| Title | An Adaptive Density Unsupervised Point Cloud Instance Segmentation Framework Based on Improved DBSCAN in Autonomous Driving Scenarios |
| Authors | Yani Zhang,Danni Hao, Kai Fan |
| Journal | Applied Soft Computing |
| Status |  Submitted  |
| Dataset | [SemanticKITTI](http://www.semantic-kitti.org/) |

---

💡 Innovations

1. Adaptive Density Estimation via k-NN Knee-Point Detection for Automatic DBSCAN Parameterization
- Proposes a k-NN distance curve knee-point detection algorithm (`_adaptive_eps_from_knee`) that adaptively determines the optimal `eps` based on local point cloud density, eliminating manual parameter tuning.
- A `sensitivity` parameter controls clustering granularity, removing the reliance on hand-crafted DBSCAN hyperparameters.
- Incorporates LiDAR intensity (`intensity_weight`) into a weighted feature space, improving discrimination between objects with different reflectance characteristics.

2. Multi-Stage Cascaded Instance Fusion Pipeline (DBSCAN → Greedy Merge → PCA Filter → Denoise)
- Coarse Clustering: Adaptive DBSCAN generates initial clusters.
- Greedy Merge (`_greedy_merge`): Multi-scale iterative centroid-based merging (1.0 m → 2.5 m) to repair over-segmentation artifacts.
- PCA Geometric Filtering (`_filter_background`): Decomposes the covariance matrix of each cluster into three geometric descriptors — `planarity`, `linearity`, and `sphericity` — to automatically reject non-object structures such as walls, poles, and spherical noise.
- Small-Cluster Removal: Filters out noise clusters with fewer than `min_cluster_size` points.

3. Grid-Based Ground Removal with Voxel Downsampling
- Grid-based ground estimation: partitions the point cloud into 2 m grid cells and uses low-percentile Z-values as local ground height.
- Employs Open3D voxel downsampling to reduce computational cost while preserving key geometric structures.
- Applies ROI cropping (`x: 0–45 m, y: ±20 m, z > −2 m`) to focus on the forward-driving scene.

4. Fully Unsupervised, Training-Free, and Label-Free
- The entire pipeline requires zero training and depends on neither deep learning models nor semantic labels.
- Naturally generalizes to unseen scenarios, making it suitable for open-world autonomous driving deployment.

 5. CGA (Contrastive-Geometric-Aware) Geometric Feature Descriptor
- Extracts a 14-dimensional per-point geometric descriptor: normals, curvature, local density, relative elevation, distance to scene center, verticality, planarity, scattering, etc.
- Enhances geometric discriminability at instance boundaries.

---

🏗️ Project Structure

```
├── main.py                    # Main evaluation script (SemanticKITTI seq 08)
├── ablation_study.py          # Ablation study (5-component additive)
├── config.yaml                # Configuration file
├── environment.yml            # Conda environment (exported from Linux)
├── requirements.txt           # Pip dependency list
├── core/
│   ├── preprocess.py          # Preprocessing: ground removal + voxel downsampling + ROI
│   ├── ipdbscan_fusion.py     # Core clustering: adaptive DBSCAN + greedy merge + PCA filter
│   └── cga_engine.py          # CGA geometric feature extractor (14-dim)
├── data/
│   └── semantic_kitti.py      # SemanticKITTI dataset loader
├── utils/
│   ├── metrics.py             # Evaluation metrics: F1 / AP@0.5 / mIoU / RI
│   └── config_loader.py       # Configuration file loader
├── plots/                     # Visualizations & ablation study results
├── results/                   # Metric CSV/Excel files & PLY visualizations
└── README.md
```

---

🚀 Quick Start

Environment Setup

```bash
# Create environment from environment.yml
conda env create -f environment.yml
conda activate ipdbscan

# Or install via pip
pip install -r requirements.txt
```

Core Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| Python | 3.10 | — |
| NumPy | 1.23 | Numerical computing |
| scikit-learn | 1.2.2 | DBSCAN / KNN / metrics |
| Open3D | 0.19.0 | Point cloud processing / voxel downsampling |
| SciPy | 1.13 | KDTree / Hungarian matching |
| PyTorch | 2.7.1+cu118 | Deep learning (PointNet++) |
| HDBSCAN | 0.8.33 | Baseline comparison |
| Matplotlib | 3.6.1 | Visualization |

Run Evaluation

1. Update the dataset path in `config.yaml`:

```yaml
data_root: "/your/path/to/semantickitti/data"
sequence: "08"           # Validation sequence
```

2. Run the main script:

```bash
python main.py
```

3. Run ablation study:

```bash
python ablation_study.py
```

Output

- Metric Report: terminal output of F1 / Precision / Recall / AP@0.5 / mIoU / RI / Latency / FPS
- CSV Results: `results/final_paper_metrics.csv`
- Ablation Plots: `plots/ablation/` (heatmaps, waterfall charts, per-class mIoU, etc.)

---

⚙️ Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `voxel_size` | 0.08 | Voxel downsampling size (m) |
| `ground_dist_threshold` | 0.24 | Ground height threshold (m) |
| `eps_base` | 0.62 | Base DBSCAN neighborhood radius |
| `min_samples` | 8 | Minimum neighbors for DBSCAN core points |
| `sensitivity` | 2.23 | Knee-point detection sensitivity |
| `min_cluster_size` | 13 | Minimum cluster size |
| `intensity_weight` | 0.15 | Intensity feature weight |

---

📊 Pipeline Overview

```
Raw Point Cloud (.bin)
    │
    ▼
┌──────────────────────┐
│  1. Preprocessing    │
│  - Voxel downsample  │
│  - Grid ground removal│
│  - ROI crop          │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  2. Adaptive Clustering│
│  - k-NN knee method  │  ← auto-select eps
│  - DBSCAN clustering │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  3. Instance Fusion  │
│  - Greedy centroid merge│
│  - PCA geometric filter│
│  - Small-cluster removal│
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  4. Evaluation       │
│  - Hungarian matching│
│  - F1 / mIoU / AP   │
└──────────────────────┘
```

---

📁 Experimental Results

| Method | F1 | mIoU | AP@0.5 | FPS |
|--------|-----|------|--------|-----|
| Raw DBSCAN | — | — | — | — |
| HDBSCAN | — | — | — | — |
| IPDBSCAN (Ours) | — | — | — | — |

> Full experimental data available in `results/baseline/` and `plots/`.

---

📜 License

[MIT License]

---

📧 Contact

[2310211226@hbut.edu.cn]
