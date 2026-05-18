# Systems Biology Project 2 — Spatiotemporal ERK Signal Propagation

**Authors:** Martyna Pawlak (459438), Aleksandra Pawłowska (459439)  
**Course:** Systems Biology 2025/26  

---

## Overview

This project analyses spatiotemporal ERK signal propagation in live MCF10A cells carrying PI3K-AKT pathway mutations. We use a graph-based model to quantify whether a cell is more likely to show a large ERK activation event if its spatial neighbours are already activating.

**Dataset:** Single-cell fluorescence microscopy tracking data from 6 experiments across 5 genotypes (WT, AKT1_E17K, PIK3CA_E545K, PIK3CA_H1047R, PTEN_del), acquired at 5 min/frame over 24 h.

---

## Repository Structure

```
biolsys_project2/
├── data/
│   └── 01-readme-experiment-description_2022-04-05.csv   # site-to-mutation metadata
├── notebooks/
│   ├── TaskA2_LaggedExposure.ipynb                       # Part A, Task 2
│   ├── TaskA3_ParameterRobustness.ipynb                  # Part A, Task 3
│   └── TaskB_BiosensorComparison.ipynb                   # Part B
├── outputs/
│   ├── A2_lagged_RR.png                                  # Figure: lagged RR curves
│   ├── A2_summary_table.csv                              # τ* and max RR per mutation
│   ├── A3_parameter_robustness.png                       # Figure: OAT sensitivity
│   ├── A3_parameter_robustness.csv                       # Numerical sweep results
│   ├── B_biosensor_comparison.png                        # Figure: ERK vs AKT RR
│   └── B_biosensor_summary.csv                           # Numerical comparison
├── docs/
│   └── Spatiotemporal_Project_2026.pdf                   # Project specification
├── spatiotemporal-continuation/                          # Provided analysis scripts
│   └── spatiotemporal-continuation/
│       ├── scripts/
│       │   ├── spatiotemporal_signal_propagation.py      # Core pipeline (single block)
│       │   └── compare_spatiotemporal_behavior.py        # Multi-block comparison
│       ├── requirements.txt
│       └── setup_env.sh
├── single-cell-tracks_exp1-6_noErbB2.csv.gz             # Raw data (655 MB, git-ignored)
└── .gitignore
```

---

## Setup

### 1. Create virtual environment

```bash
cd spatiotemporal-continuation/spatiotemporal-continuation
bash setup_env.sh
```

This creates `.venv`, installs all dependencies, and registers the Jupyter kernel `Python (SysBiol Project 2)`.

### 2. Download the data

The raw tracking data is too large for git (655 MB). Download it with:

```python
import gdown
gdown.download(
    "https://drive.google.com/uc?id=1Bx4P1mMV-zTKPDv5bNH2T4ux8BPFjF4g",
    "single-cell-tracks_exp1-6_noErbB2.csv.gz"
)
```

Place the file at the repo root (`biolsys_project2/`).

### 3. Run notebooks

```bash
jupyter notebook
```

Select kernel **Python (SysBiol Project 2)** and run notebooks in `notebooks/` top-to-bottom.

---

## Analysis Tasks

### Part A

#### Task A1 — Multi-mutation RR comparison
Compares pooled Relative Risk across all 5 genotypes using the provided `compare_spatiotemporal_behavior.py` script (parameters: r=60, W=3 frames, q=0.90). Key result: PIK3CA_H1047R shows dramatically elevated spatiotemporal coupling (RR ≈ 3.20).

#### Task A2 — Lagged exposure analysis (`TaskA2_LaggedExposure.ipynb`)
Evaluates RR(τ) for lags τ ∈ {0…6} frames (0–30 min) for WT, AKT1_E17K, and PTEN_del across 72 experiment-site blocks. Key result: τ* = 0 for all genotypes; oncogenic mutations reduce, not increase, spatial coordination strength.

#### Task A3 — Parameter robustness (`TaskA3_ParameterRobustness.ipynb`)
One-at-a-time sensitivity analysis sweeping spatial radius r ∈ {30, 60, 90, 150}, future window W ∈ {1, 3, 6, 12} frames, and jump quantile q ∈ {0.70, 0.80, 0.90, 0.95}. Key result: PIK3CA_H1047R is robustly highest across most parameter ranges, with rank inversions only at extreme values (r=30, W=12 frames).

### Part B — ERK vs AKT biosensor comparison (`TaskB_BiosensorComparison.ipynb`)

**Research question:** Does spatiotemporal signal coupling differ between the ERK biosensor (ERKKTR_ratio) and the AKT biosensor (FoxO3A_ratio), and does the mutation ranking depend on which biosensor is used?

The full analysis pipeline (jump detection, spatial graph, pooled RR) is repeated for `FoxO3A_ratio` using the same baseline parameters (r=60, W=3, q=0.90) across all 5 genotypes and all 6 experiments. Results are compared side-by-side with the ERK analysis from Task A1.

Key results:
- ERK and AKT coupling show distinct mutation-specific profiles
- PIK3CA_H1047R retains elevated coupling in both biosensors, but the magnitude and ranking among the remaining mutations differ between the two signals

---

## Key Parameters (baseline)

| Parameter | Value | Meaning |
|-----------|-------|---------|
| `spatial_radius` | 60 | Neighbourhood radius (image units) |
| `future_window_frames` | 3 (15 min) | Look-ahead window for response labelling |
| `jump_quantile` | 0.90 | Percentile threshold for jump detection |
| `signal_col` | `ERKKTR_ratio` | ERK biosensor (Part A); `FoxO3A_ratio` for Part B |
